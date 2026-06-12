# Incremental POC Plan: Multimodal OSINT Intelligence Chatbot
### Architecture Comparison, Phased Roadmap, and Technology Stack (June 2026)

**Role frame:** Principal AI Architect / Applied AI Research Engineer
**Scope:** Already-acquired multimodal corpus (HTML, PDF, text, images, screenshots, video, audio). Data acquisition out of scope.
**Companion document:** This plan operationalizes the production-architecture research in *"Designing a Production-Grade Multi-TB OSINT Intelligence Chatbot (2025–2026)"*; that report covers the target end-state, this one covers how to get there incrementally with evidence-graded choices.

**Epistemic labels used throughout:**
- **[FACT]** — published paper, official documentation, or reproducible benchmark.
- **[CONSENSUS]** — broadly agreed engineering practice across multiple independent practitioner sources.
- **[INFERENCE]** — reasoned conclusion from public signals (job posts, patents, latency behavior, blog hints) about closed systems.
- **[SPECULATION]** — plausible but weakly evidenced.

---

## 1. Executive Summary

Every leading AI search/intelligence system studied (Perplexity, Gemini Deep Research, Claude's search products, Glean, Palantir AIP, Microsoft GraphRAG) converges on the same macro-pattern: **hybrid lexical+dense retrieval → cross-encoder reranking → structured/graph context where relationships matter → agentic orchestration with explicit verification and citation enforcement** [CONSENSUS]. None of them is "a vector database with an LLM on top."

For your POC, the evidence supports this strategy:

1. **Start with hybrid retrieval + reranking on text, not naive vector RAG.** The marginal effort over basic RAG is days, and benchmarks consistently show +20–40% recall improvements (RRF fusion + cross-encoder rerank) — the single best ROI in the entire stack [FACT/CONSENSUS].
2. **Go visual-native for documents early.** For OSINT corpora full of scanned PDFs, screenshots, and layout-heavy documents, ColPali/ColQwen-style page-as-image retrieval beats OCR pipelines on exactly the documents OSINT cares about (scans, financial documents, slides); on ViDoRe-class benchmarks ColQwen2.5 reaches ~84% recall on financial PDFs vs ~62% for dense text retrieval, and OCR-error-laden hybrids can underperform vision-only retrieval on scanned material [FACT, ViDoRe leaderboard 2026].
3. **Use LazyGraphRAG/LightRAG-style graph augmentation, not full Microsoft GraphRAG, for the POC.** LazyGraphRAG indexes at ~0.1% of full GraphRAG cost with equal-or-better answer quality in Microsoft's own 96/96-win BenchmarkQED evaluation; LightRAG/HippoRAG-2 achieve similar quality at 10–30× lower cost open-source [FACT, Microsoft Research 2025; BenchmarkQED].
4. **Add agentic routing and verification last, as thin layers over a solid retrieval core.** Self-RAG/CRAG-style reflection and chain-of-verification reliably cut hallucinations (Self-RAG ~80% factuality on bio benchmarks; CoVe halves hallucination rates on list-type questions), but they amplify a good retriever — they cannot rescue a bad one [FACT + CONSENSUS].

**Bottom-line recommendations (detailed in §4):**

| Question | Answer |
|---|---|
| Simplest viable | Hybrid Search + Reranker (Phase 2 stack) |
| Highest quality | Agentic Multimodal GraphRAG with verification (Phase 6 stack) |
| Best cost/performance | Hybrid + rerank + ColPali for visual docs + LazyGraphRAG-style on-demand graph |
| Best long-term production | Modular agentic platform: hybrid+graph retrieval services, router, verifier, evaluation loop |

---

## 2. How the Leading Systems Likely Work (Condensed Evidence Map)

Your companion report covers these in depth; here is the compressed evidence map driving POC decisions.

| System | Key publicly-evidenced mechanisms | Epistemic status |
|---|---|---|
| **Perplexity** | Own crawl/index; hybrid lexical+semantic retrieval; heavy reranking; per-sentence citations; fine-tuned in-house + frontier models; query decomposition for Pro/Deep Research | [INFERENCE] from engineering blogs, job postings, observable behavior; citation mechanics [FACT] from product |
| **Gemini Deep Research** | Long-horizon agentic planner; iterative search-read-reason loops; RL-trained multi-step search; plan shown to user for approval; asynchronous task manager | [FACT] from Google's published descriptions; internals [INFERENCE] |
| **Claude (search/research)** | Agentic search with query decomposition; citation API with passage-level grounding; contextual retrieval (prepending chunk context, −67% retrieval failures per Anthropic's published numbers); parallel subagent research (published multi-agent research system blog) | [FACT] for contextual retrieval and multi-agent blog; rest [INFERENCE] |
| **Glean** | Hybrid retrieval over connectors; knowledge graph of people/docs/activity; learning-to-rank with usage signals; per-user permission-aware indexes; relevance post-mortems | [FACT/CONSENSUS] from Glean engineering blog |
| **Palantir AIP** | Ontology-first: entities/links/actions modeled before LLM; LLM as orchestrator over ontology tools; strict provenance and write-back guardrails | [FACT] from public AIP docs; ontology-as-retrieval-backbone [CONSENSUS] |
| **Microsoft GraphRAG family** | Fully published: entity/relation extraction → Leiden communities → hierarchical summaries → global/local/DRIFT search; LazyGraphRAG defers all summarization to query time at 0.1% index cost | [FACT] — papers + open source |

**Transferable design principles** [CONSENSUS]: (1) retrieval quality dominates generation quality; (2) citations are enforced structurally, not requested politely; (3) graphs/ontologies earn their cost only for relationship-and-aggregation queries; (4) agentic loops are budgeted (bounded iterations, relevance tests) rather than open-ended; (5) evaluation is continuous, with failed-query post-mortems.

---

## 3. Architecture Comparison: Eight Approaches

Scoring rubric: *Expected retrieval accuracy* = recall@k / nDCG on heterogeneous corpora relative to alternatives; *Answer accuracy* = end-to-end factual correctness with citations; *Hallucination resistance* = structural defenses, not model luck. *Confidence* = my confidence that the stated assessment is correct for an OSINT corpus of this shape, given available evidence.

---

### 3.1 Traditional (Naive) RAG

**Architecture.** Fixed-size chunking → single dense embedding model → vector DB (ANN) → top-k chunks stuffed into prompt → single LLM generation. One retrieval pass, no reranking, no routing.

**Advantages.** Days to build; cheapest per query; fully debuggable; enormous tooling ecosystem; establishes baseline metrics everything else is judged against.

**Disadvantages.** Single-vector semantic match misses exact identifiers (names, hashes, plates, phone numbers) that dominate OSINT queries — BM25 still beats dense-only retrieval on many metrics for entity/number-heavy corpora [FACT, multiple 2025–26 benchmarks]. No defense against retrieving plausible-but-wrong chunks; LLM hallucinates over gaps. Blind to images/scans/audio. Cannot answer aggregation questions ("main themes across this leak").

**Complexity & cost.** Trivial. POC: one engineer-week. Cost: embedding pass over corpus + pennies/query.

**Scalability.** Vector DBs scale to billions of vectors [FACT], but answer quality degrades as corpus grows (more distractors, no precision controls) [CONSENSUS].

**POC suitability: HIGH (as baseline only). Production: LOW.**

**Expected accuracy.** Retrieval recall@5 typically 0.45–0.65 on heterogeneous corpora; end-to-end answer accuracy 55–70% on factoid sets, far worse on multi-hop and global questions [CONSENSUS from BEIR-style and RAG benchmark literature].

**Hallucination resistance: LOW.** No verification; context gaps invite confabulation.

**Confidence: 95%.** This is the best-characterized architecture in the literature.

**Sources.** Lewis et al., RAG (2020) [FACT]; BEIR benchmark (Thakur et al. 2021) [FACT]; BM25-vs-dense comparisons in 2025–26 retrieval benchmarks [FACT]; universal practitioner consensus that naive RAG underperforms hybrid [CONSENSUS].

---

### 3.2 Hybrid Search (BM25 + Dense, RRF Fusion)

**Architecture.** Parallel lexical retrieval (BM25/SPLADE in OpenSearch/Elastic) and dense ANN retrieval; results fused via Reciprocal Rank Fusion or learned weighted scores; optional early metadata filters (time, language, source type).

**Advantages.** Captures both exact-match (critical for OSINT identifiers, aliases, codenames) and semantic similarity. RRF is parameter-light and robust. Benchmarks: +8–30% Recall@5 over BM25-only and +17–40% over dense-only [FACT, multiple published evaluations incl. your report's sources]. Anthropic's contextual retrieval work reports hybrid (contextual embeddings + contextual BM25) cutting retrieval failure rates substantially (−49% before reranking) [FACT].

**Disadvantages.** Two indexes to operate and keep consistent; fusion weights can need tuning per query class; still single-shot and text-only; still no answer verification.

**Complexity & cost.** Low-moderate. OpenSearch alone can serve both BM25 and ANN for a POC. ~1–2 weeks incremental.

**Scalability: EXCELLENT.** Both index families are proven at multi-TB scale (this is literally how web search works) [FACT].

**POC suitability: VERY HIGH. Production: HIGH (as the retrieval substrate of any final system).**

**Expected accuracy.** Recall@5 0.65–0.80 typical; answer accuracy 65–78% on factoid sets. Still weak on multi-hop/global.

**Hallucination resistance: LOW-MODERATE.** Better grounding via better recall, but no structural checks.

**Confidence: 95%.** Among the strongest-evidenced wins in IR.

**Sources.** RRF (Cormack et al. 2009) [FACT]; BEIR; Anthropic Contextual Retrieval blog (2024) [FACT]; Elastic/Weaviate/Qdrant hybrid benchmark posts [CONSENSUS]; your report's hybrid-retrieval benchmark citations [FACT].

---

### 3.3 Multi-Stage Retrieval (Candidate Generation → Rerank → Compress)

**Architecture.** Stage 1: cheap, high-recall candidate generation (hybrid, top 200–500). Stage 2: cross-encoder or listwise reranker (BGE-reranker-v2, Jina Reranker v3, Cohere Rerank) scores query–passage pairs jointly, keeps top 20–40. Optional Stage 0: lightweight SLM relevance grader to prune. Stage 3: context assembly/compression (dedup, per-source grouping, contextual headers). This is the recall-then-precision funnel used by every production search system [CONSENSUS].

**Advantages.** Rerankers add large precision gains for small compute: Anthropic reports contextual retrieval + reranking at −67% retrieval failures vs baseline [FACT]; Jina Reranker v3 hits 61.94 nDCG@10 on BEIR at 0.6B params [FACT]. Decouples recall problems from precision problems — debuggable per stage. Reduces tokens to the LLM (cost ↓, attention focus ↑).

**Disadvantages.** Adds 50–300 ms latency per query for the reranker; GPU helpful at scale; another model to version and evaluate.

**Complexity & cost.** Moderate-low. A few days on top of hybrid search; rerankers are drop-in.

**Scalability: EXCELLENT** — the funnel shape exists precisely to scale.

**POC suitability: VERY HIGH. Production: ESSENTIAL** (industry default) [CONSENSUS].

**Expected accuracy.** Recall@5-equivalent precision uplift puts answerable-question accuracy at 75–85%; this stack plus good chunking is roughly the "Perplexity-class baseline" for single-hop questions [INFERENCE].

**Hallucination resistance: MODERATE.** Cleaner context → fewer spurious generations, but still no claim-level checking.

**Confidence: 93%.**

**Sources.** ColBERT/ColBERTv2 (Khattab & Zaharia 2020; Santhanam et al. 2022) [FACT]; BGE/Jina reranker model cards + BEIR numbers [FACT]; Anthropic contextual retrieval numbers [FACT]; ubiquitous two-stage design in Glean/Perplexity engineering material [INFERENCE/CONSENSUS].

---

### 3.4 RAPTOR (Recursive Abstractive Hierarchical Indexing)

**Architecture.** Bottom-up tree: leaf chunks → cluster (GMM/UMAP) → LLM-summarize each cluster → re-embed summaries → repeat to root. Retrieval searches across all tree levels ("collapsed tree"), letting queries match abstract summaries or raw leaves.

**Advantages.** Directly attacks the "answer is spread across a document/corpus" problem; SOTA results on QuALITY/long-document QA at publication (+20% absolute with GPT-4 on QuALITY) [FACT, Sarthi et al. 2024]. Conceptually clean; integrates into existing vector stores (summaries are just more vectors).

**Disadvantages.** Index-time LLM summarization cost grows with corpus; tree goes stale on updates (incremental maintenance is awkward) [CONSENSUS]; in Microsoft's BenchmarkQED head-to-heads, RAPTOR was outperformed by LazyGraphRAG across query classes [FACT]; summaries can launder errors into higher levels with provenance loss unless citations are tracked per summary node.

**Complexity & cost.** Moderate build, but **index cost is the issue**: one LLM call per cluster per level over the whole corpus. For multi-TB, this is significant; for a POC subset, fine.

**Scalability: MODERATE.** Rebuild cost and update-staleness are the binding constraints.

**POC suitability: MODERATE** (worth testing on one document-heavy slice). **Production: NICHE** — largely superseded by GraphRAG-family hierarchical summaries and LazyGraphRAG's deferred summarization for global queries [CONSENSUS, 2026].

**Expected accuracy.** Strong on long-document and thematic questions (+10–20% over naive RAG on those classes [FACT]); little benefit on entity lookups.

**Hallucination resistance: MODERATE** for global questions (summaries provide grounded abstractions), but summary-level errors are a real failure mode.

**Confidence: 85%.** Paper results solid; production-relevance assessment is consensus-plus-inference.

**Sources.** Sarthi et al., RAPTOR (ICLR 2024) [FACT]; BenchmarkQED comparisons (Microsoft Research 2025) [FACT]; practitioner reports on staleness/cost [CONSENSUS].

---

### 3.5 GraphRAG (incl. LightRAG, LazyGraphRAG, HippoRAG-2)

**Architecture.** *Full GraphRAG (Microsoft):* LLM extracts entities/relations per chunk → knowledge graph → Leiden hierarchical community detection → LLM community summaries at multiple levels → query modes: **global** (map-reduce over community summaries), **local** (entity neighborhood + associated text units), **DRIFT** (hybrid). *LightRAG:* skips communities; dual-level (entity + relation keyword) retrieval over a lighter graph; ~1/100th index cost. *LazyGraphRAG:* builds only a noun-phrase/co-occurrence graph at index time (cost ≈ vector RAG, ~0.1% of full GraphRAG), defers extraction/summarization to query time with a relevance-test budget. *HippoRAG-2:* Personalized PageRank over an LLM-extracted graph.

**Advantages.** The only family that handles **global/sensemaking queries** ("dominant narratives across this archive", "how are these orgs connected") — vector RAG structurally cannot aggregate corpus-wide [FACT, Edge et al. 2024]. Multi-hop paths give explainability ("A → director of X → shareholder of Y → B") — exactly the OSINT product surface. LazyGraphRAG won 96/96 BenchmarkQED comparisons vs vector RAG, RAPTOR, LightRAG, and full GraphRAG modes, at vector-RAG index cost and ~700× lower query cost than GraphRAG global search [FACT, Microsoft Research 2025]. Graph doubles as the substrate for link analysis, timelines, centrality — OSINT analytics beyond chat.

**Disadvantages.** *Full* GraphRAG indexing is expensive ($20–40 per 1M tokens with GPT-4o-class models; historical large-corpus runs in the tens of thousands of dollars) [FACT]; entity drift ("Sagar S" / "S. Shankaran" as separate nodes) requires an entity-resolution pass [CONSENSUS]; incremental updates are awkward in full GraphRAG; graph quality is hostage to the extraction prompt; point lookups are better served by plain hybrid retrieval, so a **router** is needed [CONSENSUS].

**Complexity & cost.** High for full GraphRAG; **low-moderate for LightRAG/LazyGraphRAG** — this is the decisive 2025–26 development. LightRAG indexes ~$0.50 per 1M tokens vs $20–40 [FACT, practitioner benchmarks].

**Scalability.** Graph DBs (Neo4j, NebulaGraph, Memgraph) scale to billions of edges [FACT]; the LLM-extraction pass is the scaling bottleneck — mitigate with SLM extractors and LazyGraphRAG-style deferral [CONSENSUS].

**POC suitability: HIGH via LightRAG/LazyGraphRAG; LOW via full GraphRAG. Production: HIGH** for OSINT specifically — entity/relationship questions are the core workload.

**Expected accuracy.** On local/factoid queries: ≈ hybrid RAG (no harm, modest help). On multi-hop and global: large wins — GraphRAG-family methods show 2–4× preference/win-rates on comprehensiveness and diversity vs vector RAG on global query sets [FACT]; LazyGraphRAG best-in-family on the cost-quality frontier [FACT].

**Hallucination resistance: MODERATE-HIGH** for relationship claims (the graph either contains the edge with provenance or it doesn't), but extraction-time hallucinations can poison the graph — extraction verification matters.

**Confidence: 90%** for the family's value on OSINT workloads; **80%** for any specific variant ranking holding on *your* corpus (BenchmarkQED corpora ≠ noisy OSINT).

**Sources.** Edge et al., "From Local to Global" (2024) [FACT]; Microsoft LazyGraphRAG + BenchmarkQED blogs (2025) [FACT]; Guo et al., LightRAG (2024) [FACT]; HippoRAG-2 (2025) [FACT]; Neo4j/Memgraph case studies [FACT]; "graph for global, vector for local, route by query type" [CONSENSUS 2026].

---

### 3.6 Agentic RAG

**Architecture.** An LLM agent (or small team of agents) controls retrieval as a tool-use loop: query analysis → plan → decompose into sub-queries → choose retriever(s) per sub-query (hybrid index, graph, SQL/metadata, visual index, web) → read → decide whether to retrieve again → synthesize. Variants: single ReAct-style agent; router + specialist agents (retrieval agent, graph agent, verification agent, citation agent); orchestrator-worker (Anthropic's published multi-agent research architecture: lead agent spawns parallel subagents per sub-question) [FACT for Anthropic's design; CONSENSUS for the pattern].

**Advantages.** Only architecture that handles compound investigative questions ("Find connections between org X and persons in these leaks between 2019–2021, and assess source reliability") — fixed pipelines cannot adapt retrieval strategy mid-query. Query decomposition alone yields large gains on multi-hop benchmarks [FACT, e.g., self-ask / decomposition literature]. Naturally extends to tools beyond retrieval: timeline builder, geocoder, EXIF reader, translator. Deep Research products (OpenAI, Gemini, Perplexity) are all agentic loops, and they dominate hard research benchmarks vs single-shot RAG [FACT for products' published descriptions; INFERENCE for internals].

**Disadvantages.** Latency: seconds → minutes. Cost: 3–30× single-shot. Nondeterminism complicates evaluation and debugging. Agents can loop, over-search, or rationalize weak evidence; budgets and stop-conditions are mandatory [CONSENSUS]. Quality ceiling still set by the underlying retrievers — an agent over a bad index is just slower wrongness.

**Complexity & cost.** Moderate-high. Frameworks (LangGraph, Pydantic-AI, CrewAI, Semantic Kernel) reduce plumbing but not the evaluation burden.

**Scalability.** Horizontally scalable (stateless workers), but cost-per-query scales with agent steps — budget enforcement is the real scaling problem [CONSENSUS].

**POC suitability: HIGH in Phase 5 (after retrieval substrate is solid). Production: ESSENTIAL** for an investigation assistant; this is the visible product layer.

**Expected accuracy.** On multi-hop/compound questions: +15–30% over single-shot pipelines with the same retrievers [CONSENSUS from decomposition/agentic-RAG literature, e.g., Adaptive-RAG, Search-R1-style results]. On simple lookups: no gain, pure overhead — hence routing.

**Hallucination resistance: MODERATE-HIGH** when paired with verification agents; the loop can notice "insufficient evidence" and say so — single-shot RAG structurally cannot.

**Confidence: 85%** on the value; **70%** on any specific multi-agent topology being optimal (fast-moving area, weak public benchmarks for topologies).

**Sources.** ReAct (Yao et al. 2023) [FACT]; Adaptive-RAG (Jeong et al. 2024) [FACT]; Anthropic multi-agent research system blog (2025) [FACT]; Gemini/OpenAI Deep Research product docs [FACT]; Search-R1/ReSearch RL-trained search agents (2025) [FACT]; topology best practices [CONSENSUS/INFERENCE].

---

### 3.7 Reflection & Verification Pipelines (Self-RAG, CRAG, CoVe, faithfulness checking)

**Architecture.** Wraps any retrieval core with quality control loops:
- **Self-RAG:** model emits reflection tokens — retrieve? / is passage relevant? / is output supported? / is output useful — and critiques its own generation [FACT, Asai et al. 2023].
- **CRAG (Corrective RAG):** lightweight retrieval evaluator grades retrieved docs {correct / incorrect / ambiguous}; on failure, triggers query rewriting and fallback retrieval (e.g., broader index or web) [FACT, Yan et al. 2024].
- **Chain-of-Verification (CoVe):** draft → generate verification questions → answer them independently against evidence → revise [FACT, Dhuliawala et al. 2023].
- **Claim-level grounding/NLI:** post-hoc faithfulness scoring of each generated claim against cited passages (NLI models, AlignScore-style, or LLM-as-judge); unsupported claims dropped or flagged; per-claim confidence = f(#independent sources, agreement, source credibility, retrieval rank) [CONSENSUS pattern across Perplexity/Claude citation behavior; implementation FACTs vary].

**Advantages.** The biggest lever on hallucination specifically: Self-RAG reports ~80% factuality on biography generation vs much lower baselines; CoVe roughly halves hallucinations on list-form questions [FACT]. CRAG-style grading is cheap (an SLM can do it) and catches the most common failure (bad retrieval) before generation [FACT/CONSENSUS]. For OSINT, claim-level confidence + multi-source consensus is a *product requirement*, not a nicety — analysts must see why a claim is trusted.

**Disadvantages.** Not a retrieval architecture — must sit on one. Adds 1.5–3× generation cost and latency. LLM-as-judge verification has its own error rate and can rubber-stamp; NLI checkers are brittle on long, compositional claims [CONSENSUS]. Self-RAG as published requires fine-tuning for reflection tokens (most teams approximate with prompted critics) [FACT/CONSENSUS].

**Complexity & cost.** Low-moderate to bolt on prompted versions; moderate for trained graders; verification tokens are cheap if SLMs do grading and the big model only drafts/revises.

**Scalability: GOOD** — graders/verifiers parallelize and can be tiny models.

**POC suitability: HIGH (Phase 5–6). Production: ESSENTIAL for OSINT** — wrong-but-confident answers are the catastrophic failure mode in intelligence work.

**Expected accuracy.** +5–15% end-to-end answer accuracy on top of a strong retrieval stack; hallucination/unsupported-claim rates cut by 40–60% on benchmark-style evals [FACT-anchored estimate from Self-RAG/CoVe/CRAG numbers; exact transfer to OSINT is INFERENCE].

**Hallucination resistance: HIGH** — this is the layer that creates it.

**Confidence: 88%** that this layer materially reduces unsupported claims; **75%** on the magnitude transferring to noisy OSINT data.

**Sources.** Asai et al., Self-RAG (2023); Yan et al., CRAG (2024); Dhuliawala et al., CoVe (2023); Es et al., RAGAS (2023) [all FACT]; Perplexity/Claude citation-enforcement behavior [INFERENCE]; multi-source consensus scoring as OSINT practice [CONSENSUS].

---

### 3.8 Multimodal RAG Architectures

**Architecture.** Three production patterns in 2026 [CONSENSUS, confirmed by current practitioner literature]:
1. **Caption-and-index (translate-to-text):** VLM captions images/charts; ASR (Whisper-class) transcribes audio/video; OCR extracts scan text; everything indexed as text in the standard pipeline. Simplest; lossy on layout/visual detail.
2. **Unified multimodal embeddings:** one embedding space for text+images (Cohere Embed-4, voyage-multimodal-3.x, jina-clip-v2, SigLIP-2 derivatives); cross-modal nearest-neighbor search; single-vector cost profile. Now competitive with late-interaction on most *enterprise* corpora at a fraction of storage [FACT, ViDoRe-V2-era analyses].
3. **Page-as-image late interaction (ColPali / ColQwen2.5 / ColNomic / ColModernVBERT):** embed page screenshots as patch-level multi-vectors; MaxSim late interaction; no OCR/parsing at all. Strongest on visually hard retrieval — scanned docs, charts, financial tables (≈84% vs ≈62% dense-text recall on financial PDFs; OCR-based hybrids can fall *below* vision-only on scanned corpora) [FACT, ViDoRe leaderboard 2026]. Storage-heavy (~1024 patch vectors/page; mitigated by pooling/compression and two-stage search [FACT]). ViDoRe-V1 is saturated (>90 nDCG@5); ViDoRe-V2 is the meaningful benchmark now [FACT].
   Generation side: a VLM (Qwen2.5-VL/Qwen3-VL, GPT-4o/5-class, Gemini, Claude) answers over retrieved page images — with the known caveat that **VLMs hallucinate chart numbers**, so chart-QA evals and bounding-box/page citations are required [CONSENSUS].

**Advantages.** OSINT corpora are *adversarially* multimodal: screenshots of chats, scanned leaks, memes, video frames, intercepted audio. Text-only RAG silently discards most of that signal [CONSENSUS]. Pattern 3 eliminates the OCR-error cascade entirely on scans. Pattern 1 lets you reuse the entire text stack and is the correct POC entry point.

**Disadvantages.** Pattern 1 loses visual/spatial information and caption quality bounds retrieval. Pattern 3: storage/compute heavy, vector-DB support for late interaction is uneven (Vespa native; Qdrant multivector good; OpenSearch needs custom scoring) [FACT]; page-level granularity. Video/audio need their own chunking (scene/diarization). Cross-modal eval tooling is immature [CONSENSUS].

**Complexity & cost.** Pattern 1: low. Pattern 2: moderate. Pattern 3: moderate-high (GPU indexing; self-hosted ColQwen indexing is ~37× cheaper/page than managed document-intelligence APIs per current GPU-cloud analyses [FACT, single-source — treat as indicative]).

**Scalability.** Pattern 1–2 scale like text RAG. Pattern 3 scales with pooling + two-stage retrieval (coarse single-vector → late-interaction rerank) [FACT, Visual RAG Toolkit-style work].

**POC suitability: HIGH — Pattern 1 in Phase 3, Pattern 3 for the document/scan slice in Phase 3b. Production: ESSENTIAL** for this use case; recommended end-state is Pattern 1 for audio/video + Pattern 3 (or 2) for documents/images, fused at the candidate level.

**Expected accuracy.** On visual-document questions: +20–35% retrieval recall vs OCR-text-only on scan/chart-heavy slices [FACT-anchored from ViDoRe-class gaps]; on born-digital clean text: parity. End-to-end multimodal answer accuracy is corpus-dependent; budget for VLM numeric hallucination.

**Hallucination resistance: MODERATE** — retrieval grounding improves, but VLM reading errors (charts, small text) are a new hallucination channel needing verification (cross-check OCR vs VLM reads) [CONSENSUS].

**Confidence: 85%** for the architecture pattern ranking; **75%** for magnitude on your specific corpus mix.

**Sources.** Faysse et al., ColPali (ICLR 2025) + ViDoRe V1/V2 [FACT]; REAL-MM-RAG (2025) showing rephrasing fragility [FACT]; current multimodal-RAG architecture guides (BigDataBoutique 2026, GPU-cloud deployment analyses 2026) [CONSENSUS/FACT]; VisRAG/DSE [FACT]; Whisper large-v3 WER literature [FACT].

---

## 4. Recommendations

### 4.1 Simplest architecture that is still defensible
**Hybrid (BM25+dense, RRF) → cross-encoder rerank → cited generation** (the Phase 2 stack), with caption/transcribe-to-text for non-text media. One search engine (OpenSearch or Qdrant+Tantivy hybrid), one embedding model, one reranker, one LLM. Skipping the reranker to save effort is false economy — it is days of work for the largest single quality jump [CONSENSUS]. *Confidence: 92%.*

### 4.2 Highest-quality architecture (cost no object)
**Agentic multimodal Graph-RAG with verification:** router → {hybrid text retrieval | ColQwen visual retrieval | graph local/global search | timeline/metadata SQL} → cross-encoder + late-interaction reranking → orchestrator-worker agents for decomposition → draft → CoVe + claim-level NLI grounding + multi-source consensus scoring → cited, confidence-annotated answer. Frontier LLM for synthesis, SLM fleet for grading/extraction. This mirrors the converged design of Deep-Research-class systems plus Palantir-style ontology grounding [INFERENCE/CONSENSUS]. *Confidence: 85%.*

### 4.3 Best cost-performance
**Phase 2 stack + ColPali-family index for the visual-document slice + LazyGraphRAG-style deferred graph for global/relationship queries + CRAG-style SLM retrieval grader.** Rationale: hybrid+rerank captures most factoid quality at near-zero marginal cost; ColQwen only where text retrieval demonstrably fails (scans/charts); LazyGraphRAG gives GraphRAG-class global answers at vector-RAG index cost [FACT]; one SLM grader buys most of the hallucination reduction at ~10% of full verification cost. *Confidence: 88%.*

### 4.4 Best long-term production architecture
**Modular retrieval platform, not a framework monolith:** independently deployable services — ingestion (Kafka + workers), text index (OpenSearch), vector index (Milvus/Qdrant), visual index (ColQwen on Vespa/Qdrant), knowledge graph (Neo4j/NebulaGraph + entity-resolution pipeline), reranking service, model gateway (vLLM serving an LLM/SLM/VLM mix), agent orchestrator (LangGraph-class, but behind your own API), verification service, and a continuous evaluation loop (golden sets + failed-query post-mortems, Glean-style). Every leading system converged on routed heterogeneous retrieval behind an agentic front [CONSENSUS/INFERENCE]; frameworks change yearly, service boundaries don't. *Confidence: 85%.*

---

## 5. Phased POC Roadmap

Cross-phase principles: (a) **build the evaluation harness in Phase 1 and never stop running it** — every phase's "expected accuracy improvement" is only real if measured on your golden set; (b) work on a **representative 1–5% corpus slice** (stratified across formats/languages/sources) until Phase 4+; (c) keep each phase shippable — a phase is done when its eval gate passes, not when its features exist.

---

### Phase 1 — Basic RAG (Weeks 1–3)

**Goals.** End-to-end baseline; corpus reconnaissance (format/language/quality census); golden evaluation set v1 (100–150 questions: factoid, multi-doc, multi-hop, global, multimodal, unanswerable — with graded relevant docs); baseline metrics everything else is judged against.

**Features.** Text extraction for HTML (trafilatura) and born-digital PDFs (PyMuPDF/Docling); recursive ~512-token chunking with overlap + metadata (source, date, type, language); dense embeddings; vector search; top-k → LLM with mandatory inline citations; minimal chat UI; logging of every retrieval set.

**Technologies.** Docling or unstructured.io; BGE-M3 or gte-Qwen2 embeddings; Qdrant (single node); vLLM serving Qwen2.5/Qwen3-7B-class (or hosted frontier model if data permits); RAGAS + custom retrieval metrics; Langfuse/Phoenix tracing.

**Deliverables.** Working chatbot on corpus slice; corpus census report; golden set v1; baseline scorecard (Recall@5/20, nDCG@10, faithfulness, answer accuracy, unanswerable-detection rate); failure taxonomy.

**Risks.** Garbage-in from parsing (mitigate: parse-quality sampling); golden set too easy (mitigate: include known-hard multi-hop/global/visual questions even though they'll fail now — they define later phases' headroom); team skipping eval to build features.

**Expected accuracy.** Baseline, not improvement. Anticipate Recall@5 ≈ 0.45–0.6; answer accuracy ≈ 55–65% on factoid; near-zero on global/visual questions [CONSENSUS-anchored expectation].

---

### Phase 2 — Hybrid Retrieval + Reranking (Weeks 3–6)

**Goals.** Lift retrieval to production-respectable; add the recall→precision funnel; introduce contextual chunking.

**Features.** BM25 (OpenSearch) alongside dense; RRF fusion; metadata pre-filters (time/source/language); cross-encoder reranking top-200→top-20; Anthropic-style contextual chunk headers (prepend doc-level context to each chunk before embedding — published −49% retrieval-failure reduction, −67% with reranking [FACT]); query rewriting (spelling, alias expansion from a seed alias list).

**Technologies.** OpenSearch (BM25 + ANN, also your aggregations/dashboards later); keep Qdrant or consolidate into OpenSearch for POC simplicity; BGE-reranker-v2-m3 or Jina Reranker v3; SLM (Qwen3-1.7B/Phi-4-mini) for query rewriting.

**Deliverables.** Hybrid+rerank pipeline behind a retrieval API (clean service boundary — everything later calls this); ablation report (dense-only vs BM25-only vs hybrid vs hybrid+rerank on golden set); latency budget doc.

**Risks.** Fusion-weight overfitting to golden set (mitigate: held-out split); reranker latency on CPU (mitigate: small GPU or Jina/Cohere API for POC); index-consistency drift between engines.

**Expected accuracy improvement.** +15–25 pts Recall@5 (to ~0.70–0.80); answer accuracy → ~72–80% on factoid/multi-doc [FACT-anchored from hybrid+rerank literature]. This is the single largest jump in the roadmap.

---

### Phase 3 — Multimodal Processing (Weeks 6–11)

**Goals.** Stop discarding the non-text majority of the corpus; visual-document retrieval that survives scans.

**Features.**
- *3a Translate-to-text:* OCR for scans (Surya/PaddleOCR, or VLM-OCR via Qwen-VL for hard cases); Whisper large-v3 + diarization (WhisperX/pyannote) for audio and video audio tracks; keyframe extraction + VLM captioning for video/images; EXIF/metadata harvesting; all indexed into the Phase 2 pipeline with media-type metadata and timestamp anchors (audio/video answers cite mm:ss).
- *3b Visual-native retrieval:* ColQwen2.5 (or ColModernVBERT for cheap/fast) page-image index over PDFs/screenshots/scans; two-stage visual search (pooled coarse → late-interaction rerank); candidate-level fusion with text retrieval via RRF; VLM answerer over retrieved page images with page-region citations.

**Technologies.** ffmpeg + PySceneDetect; WhisperX; Qwen2.5-VL-7B (captioning + VLM-OCR + answering); ColQwen2.5 / colnomic; Qdrant multivector or Vespa for late interaction; chart-QA mini-eval set (VLMs hallucinate chart numbers — test for it explicitly [CONSENSUS]).

**Deliverables.** Multimodal ingestion DAG; visual index over document slice; fused multimodal retrieval API; golden set v2 with ≥30 visual/audio questions; report: OCR-pipeline vs ColQwen vs hybrid on the scanned-document slice (decide pattern per media type from *your* data, not blog posts).

**Risks.** ASR errors on noisy/multilingual audio (mitigate: language ID + per-language models, confidence-flag low-quality transcripts); ColQwen storage blow-up (mitigate: pooling/binary quantization; index only doc-like media); VLM chart hallucination (mitigate: cross-check VLM read vs OCR tokens; flag disagreement); scope creep — *do not* attempt face/object recognition in POC.

**Expected accuracy improvement.** No change on text questions; visual/audio questions go from ~0% answerable to 60–75%; on scanned-doc retrieval expect ColQwen ≈ +20 pts recall over OCR-text path [FACT-anchored, ViDoRe-class gaps]; overall golden-set accuracy +8–15 pts purely from newly answerable classes.

---

### Phase 4 — GraphRAG (Weeks 11–16)

**Goals.** Entity-centric and global/sensemaking answers; the link-analysis substrate OSINT analysts actually want.

**Features.** NER + relation extraction (SLM/GLiNER-class extractors with a pinned, version-controlled extraction prompt — prompt drift silently corrupts graphs [CONSENSUS]); entity resolution (embedding similarity + alias rules + deterministic keys for emails/phones/IDs); knowledge graph with per-edge provenance (every edge → source TextUnits); LightRAG-style dual-level retrieval for entity queries; LazyGraphRAG-style query-time community summarization for global queries; query router (SLM classifier): {point lookup → Phase 2 path | entity/multi-hop → graph local | global → lazy graph global | visual → Phase 3 path}; timeline view (event nodes with time/place/participants).

**Technologies.** Neo4j (POC; NebulaGraph if/when distributed) or Memgraph; GLiNER / Qwen3-4B extractor; LightRAG and/or LazyGraphRAG implementations (Microsoft's, or graphrag+lazy variants); dedupe/splink-style ER; NetworkX for analytics prototypes (centrality, communities).

**Deliverables.** KG over corpus slice with provenance + ER report (merge precision/recall on a labeled entity sample); routed retrieval API v2; golden set v3 with ≥30 multi-hop/global/timeline questions; ablation: vector-only vs graph-local vs lazy-global per query class; graph exploration UI (even rudimentary — analysts will live here).

**Risks.** Extraction-quality hostage-taking (mitigate: eval extractor on labeled sample *before* full run); entity drift (mitigate: ER pass + periodic merge audits); over-routing to the graph (mitigate: router eval; default to hybrid on low confidence); full-GraphRAG cost temptation (mitigate: LazyGraphRAG first; full community summaries only if lazy quality demonstrably insufficient on your data [FACT: lazy matched/beat full in BenchmarkQED]).

**Expected accuracy improvement.** Multi-hop: +15–30 pts; global/sensemaking: from largely unanswerable to 60–75% acceptable-answer rate [FACT-anchored from GraphRAG-family evals]; factoid: ±0 (router protects it). Overall golden set: +5–12 pts.

---

### Phase 5 — Agentic Reasoning and Verification (Weeks 16–21)

**Goals.** Compound investigative questions; structural hallucination defenses; "insufficient evidence" as a first-class answer.

**Features.** Planner agent: decompose compound queries into sub-questions with per-sub-question retriever selection; bounded ReAct loop (max iterations, token budget, relevance-test budget — LazyGraphRAG-style budgeting generalizes well [INFERENCE]); CRAG-style SLM retrieval grader gating generation (grade {correct/ambiguous/incorrect} → rewrite query / broaden / escalate on failure); citation enforcement: every sentence must carry ≥1 citation or be tagged [ANALYSIS]/[INFERENCE]-by-the-system; chain-of-verification on high-stakes answers; multi-source consensus + source-credibility scoring feeding per-claim confidence (≥2 independent sources for sensitive claims or explicit single-source flag); answer abstention path.

**Technologies.** LangGraph (or Pydantic-AI) behind your own orchestrator API; SLM fleet via vLLM (grader, router, citation-aligner: 1–4B models); frontier or strong open model (Qwen3-32B/72B-class, or hosted Claude/Gemini where data policy allows) for planning/synthesis; NLI/AlignScore-style claim checker.

**Deliverables.** Investigator mode (multi-minute deep answers) alongside fast mode; verification report per answer (claims, citations, confidence, dissenting sources); agent-trace viewer; cost/latency budget per mode; red-team eval: 50 adversarial questions (false-premise, unanswerable, conflicting-source) with measured abstention/correction rates.

**Risks.** Cost blow-up (mitigate: hard budgets; SLMs for all grading); agent loops/rationalization (mitigate: trace audits, stop-conditions); verification rubber-stamping (mitigate: verifier model ≠ generator model; spot-check with humans); latency UX (mitigate: streaming plan + progressive answers, Deep-Research-style).

**Expected accuracy improvement.** Compound/multi-hop: +10–20 pts over Phase 4; unsupported-claim rate −40–60% [FACT-anchored from Self-RAG/CoVe/CRAG, transfer is INFERENCE]; false-premise/unanswerable handling from near-random to >80% correct abstention/correction.

---

### Phase 6 — Reflection and Self-Correction (Weeks 21–26)

**Goals.** The system improves itself: failed queries become training/config signal; reflection becomes routine, not bolted on.

**Features.** Self-RAG-style reflective generation (prompted critic pass: is each segment supported? useful? — regenerate or trim if not); adaptive retrieval depth (Adaptive-RAG: classify query complexity → no-retrieval / single-shot / iterative) [FACT]; failed-query post-mortem loop (Glean-style): weekly triage of thumbs-down/low-confidence traces → labeled additions to golden set → targeted fixes (chunking, router, extraction prompt, fusion weights); optional fine-tunes from accumulated traces: SLM relevance grader (published precision jumps from ~0.13→~0.77 after fine-tuning graders [FACT, per companion report's source]), router, citation aligner; embedding/reranker domain adaptation if eval shows headroom; drift monitors (retrieval-score distributions, abstention rates, per-source credibility updates).

**Technologies.** Langfuse/Phoenix + custom eval dashboards; RAGAS + DeepEval + your golden sets in CI (eval-gated deploys); LoRA/QLoRA fine-tuning for SLM roles only (avoid fine-tuning the main generator [CONSENSUS]); Ray for batch eval/re-embedding jobs.

**Deliverables.** Continuous-eval pipeline in CI; post-mortem playbook + first three completed post-mortem cycles; fine-tuned grader v1 with before/after ablation; POC final report: full scorecard trajectory Phase 1→6, cost per query per mode, production-scaling plan (the companion report's multi-TB architecture becomes the blueprint).

**Risks.** Eval-set overfitting (mitigate: rotating held-out sets); feedback loops amplifying biased source-credibility scores (mitigate: human review of credibility changes); fine-tuning regressions (mitigate: eval gates).

**Expected accuracy improvement.** +3–8 pts overall (diminishing returns — this phase's value is *slope*, not a step: the system now improves monthly); grader fine-tunes can add outsized retrieval-precision gains where the base grader was weak [FACT-anchored].

---

## 6. Tool Recommendations and Justifications

Confidence = confidence this is the right *POC* choice for this use case in mid-2026.

### 6.1 Parsing & multimodal understanding
**Pick: Docling (IBM) primary; unstructured.io fallback; Qwen2.5-VL for hard layouts/VLM-OCR.**
Docling: best open-source layout/table fidelity per current comparisons, permissive license, active development; unstructured.io has broader format coverage but weaker tables; commercial parsers (LlamaParse, Azure Doc Intelligence) are strong but costly at corpus scale (self-hosted VLM parsing runs order-of-magnitude cheaper per page [FACT, single-source]). Cons: Docling slower than naive PyMuPDF — use PyMuPDF for born-digital fast path, Docling for complex docs. Scalability: parallelize on Ray. *Confidence: 80%* (parser rankings churn quarterly).

### 6.2 OCR & speech-to-text
**Pick: Surya or PaddleOCR for bulk OCR; Qwen-VL OCR for degraded scans; WhisperX (large-v3) + pyannote diarization for audio.**
Whisper large-v3 remains the open multilingual ASR default; WhisperX adds word-level timestamps (citation anchors) and diarization (speaker attribution — OSINT-critical). Cons: weak on heavy code-switching/noisy intercepts — confidence-flag and route to bigger models. Alternatives: NVIDIA Canary/Parakeet (faster, fewer languages), hosted ASR (cost + data-policy issues). *Confidence: 85% (ASR), 75% (OCR — VLM-OCR is eating classical OCR; revisit at Phase 3).*

### 6.3 Embeddings & multimodal embeddings
**Pick: BGE-M3 (text, POC) → evaluate gte-Qwen / NV-Embed-class for upgrade; ColQwen2.5 (visual docs); jina-clip-v2 or SigLIP-2 (photos/memes); whisper-transcript text embeddings for audio.**
BGE-M3: multilingual, 8k context, dense+sparse+multi-vector from one model (simplifies hybrid), permissive. Top-of-MTEB 7B-class embedders score higher but cost 5–10× to run over a large corpus — wrong POC trade [CONSENSUS]. ColQwen2.5: ViDoRe-leading family, good ecosystem. Cons: MTEB leadership churns; mitigate by versioning embeddings and keeping re-embedding cheap (Ray batch). *Confidence: 85% (text), 80% (visual).*

### 6.4 Vector database
**Pick: Qdrant for POC; Milvus for production scale; Vespa if late-interaction becomes central.**
Qdrant: fastest to operate, native multivector (ColPali-friendly), good filtering, single binary. Milvus: best-proven at billions of vectors, GPU indexes, but heavier ops — production step. Vespa: native ColBERT-style scoring + lexical + tensor math in one engine — the technically cleanest late-interaction home, but steepest learning curve. pgvector: fine under ~50M vectors, simplest ops if team is Postgres-native. *Confidence: 85%.*

### 6.5 Graph database
**Pick: Neo4j (POC and likely production); NebulaGraph if horizontally-distributed writes become necessary; Memgraph for hot in-memory analytics subgraphs.**
Neo4j: richest ecosystem (GraphRAG integrations, GDS algorithms for centrality/communities, visualization tooling, the de-facto GraphRAG reference stack [CONSENSUS]). Cons: clustering costs (enterprise) — but a POC graph fits one node easily. Apache AGE on Postgres works for small graphs if minimizing components. *Confidence: 85%.*

### 6.6 Search engine
**Pick: OpenSearch.**
BM25 + ANN + aggregations + dashboards in one Apache-2.0 system; also serves analyst-facing faceting/time-histograms later. Elasticsearch is equivalent tech with licensing considerations; Tantivy/Lucene-embedded options (or Qdrant's sparse vectors) are fine if consolidating components for the POC. *Confidence: 90%.*

### 6.7 Rerankers
**Pick: BGE-reranker-v2-m3 (cheap default) and Jina Reranker v3 (quality, listwise, 61.94 BEIR nDCG@10 at 0.6B [FACT]); optional fine-tuned SLM grader (Phase 6).**
Cross-encoders are the consensus precision layer. Cohere Rerank 3.5 is the strongest managed option if data policy allows. Cons: GPU for low latency at scale. *Confidence: 90%.*

### 6.8 LLMs & SLMs
**Pick: tiered fleet via vLLM —**
- **Synthesis/planning:** Qwen3-32B-class open-weight (or hosted Claude/Gemini/GPT where data policy allows — for sensitive OSINT, assume self-hosted).
- **Workhorse generation:** Qwen3-7/8B-class or Llama-3.x-8B.
- **SLM roles (router, grader, citation-aligner, extractor):** Qwen3-1.7B/4B, Phi-4-mini, GLiNER for NER.
- **VLM:** Qwen2.5-VL-7B (answering + captioning + OCR fallback).
Rationale: role-tiered fleets are how production systems control cost [CONSENSUS]; Qwen family gives consistent tokenizer/behavior across sizes + VLM, strong multilingual (OSINT-relevant), permissive license. Cons: open models trail frontier on hardest synthesis — that's what the optional hosted tier is for. INT4/AWQ quantization for SLM roles. *Confidence: 80% (model names churn; the tiering pattern is the durable recommendation).*

### 6.9 Agent frameworks
**Pick: LangGraph behind your own orchestrator API; Pydantic-AI as the lighter alternative.**
LangGraph: explicit state-machine graphs (debuggable, resumable, budget-enforceable — exactly what bounded agentic RAG needs), largest ecosystem. CrewAI/AutoGen: better for role-play multi-agent demos, weaker production controls [CONSENSUS]. Semantic Kernel if .NET shop. Key justification: *wrap whatever framework in your own service boundary* — framework churn is the highest-velocity risk in the stack. *Confidence: 75%.*

### 6.10 Evaluation frameworks
**Pick: RAGAS (RAG metrics: faithfulness, context precision/recall) + DeepEval (CI-style assertions) + Langfuse or Arize Phoenix (tracing, datasets, human annotation queues) + custom golden-set harness.**
No single framework suffices [CONSENSUS]; the golden set + failed-query post-mortems are the actual quality engine — frameworks are plumbing. LLM-as-judge metrics need periodic human calibration (judge error is real [CONSENSUS]). *Confidence: 85%.*

---

## 7. System Architecture, Data Flow, and Query Flow per Phase

### Phase 1 — Basic RAG
```
INGEST:  HTML/PDF/TXT ─► Parser (Docling/trafilatura) ─► Chunker(512tok+meta)
         ─► Embedder (BGE-M3) ─► Qdrant
QUERY:   user q ─► embed ─► ANN top-k ─► prompt(q + chunks) ─► LLM ─► cited answer
         └────────────── all traces → Langfuse → eval harness ──────────────┘
```

### Phase 2 — Hybrid + Rerank
```
INGEST:  parser ─► contextual chunker (doc-context header per chunk)
         ├─► BM25 index (OpenSearch)
         └─► dense index (Qdrant/OpenSearch ANN)
QUERY:   q ─► SLM rewrite/alias-expand ─► [BM25 top300 ∥ ANN top300]
         ─► metadata filters ─► RRF fuse (~150) ─► cross-encoder rerank (top20)
         ─► context assembly (dedup, group-by-source) ─► LLM ─► cited answer
```

### Phase 3 — Multimodal
```
INGEST:  router by MIME ──► docs/scans ─► Docling/OCR ─► text path
                        ├─► docs/scans ─► page renders ─► ColQwen2.5 ─► multivector index
                        ├─► images     ─► VLM caption + EXIF ─► text path (+CLIP-style index)
                        ├─► audio      ─► WhisperX + diarize ─► timestamped chunks ─► text path
                        └─► video      ─► audio track→ASR; keyframes→VLM captions ─► text path
QUERY:   q ─► modality-aware retrieval: [hybrid text ∥ ColQwen visual ∥ image index]
         ─► RRF fuse across modalities ─► rerank ─► LLM/VLM (page images attached if visual)
         ─► answer with page/timestamp citations
```

### Phase 4 — + GraphRAG
```
INGEST:  text path ─► NER/RE extractor (pinned prompt) ─► entity resolution
         ─► KG (Neo4j: entities, relations, events; per-edge provenance→TextUnits)
QUERY:   q ─► SLM ROUTER ──► point lookup ──► Phase 2/3 path
                        ├─► entity/multi-hop ─► graph local (k-hop + linked TextUnits) ─► rerank ─► LLM
                        ├─► global/theme ─► LazyGraphRAG (query-time relevance-budgeted
                        │                   community build + map-reduce summarize)
                        └─► timeline ─► event-node sort + evidence ─► LLM
         all paths ─► cited answer (+ path explanation for graph answers)
```

### Phase 5 — + Agentic & Verification
```
q ─► PLANNER (decompose; pick retriever per sub-q; set budgets)
   ─► loop ≤N: retrieve ─► CRAG GRADER (SLM): correct? ─► no: rewrite/broaden/escalate
   ─► evidence pool ─► SYNTHESIZER (draft, every sentence cited or tagged [ANALYSIS])
   ─► VERIFIER: CoVe questions + NLI claim-vs-citation + multi-source consensus
   ─► revise/trim/abstain ─► answer + per-claim confidence + dissent notes + trace
```

### Phase 6 — + Reflection & Self-Correction (steady state)
```
runtime:  Phase 5 flow + Adaptive-RAG complexity classifier (skip/single/iterative retrieval)
          + reflective critic pass on each answer segment
offline:  traces + feedback ─► weekly post-mortems ─► golden-set growth ─► CI eval gates
          ─► targeted fixes + LoRA fine-tunes (grader/router/aligner) ─► redeploy
```

---

## 8. Final Comparison Tables

### 8.1 Architecture scorecard

| Approach | Retrieval acc. | Answer acc. | Halluc. resist. | Complexity | Cost (build/run) | Scalability | POC fit | Prod fit | Conf. |
|---|---|---|---|---|---|---|---|---|---|
| Traditional RAG | ★★ | ★★ | ★ | ★ (trivial) | $ / $ | ★★★ | baseline only | ✗ | 95% |
| Hybrid Search | ★★★★ | ★★★ | ★★ | ★★ | $ / $ | ★★★★★ | ✓✓ | substrate | 95% |
| Multi-stage (rerank) | ★★★★★ | ★★★★ | ★★★ | ★★ | $ / $$ | ★★★★★ | ✓✓ | essential | 93% |
| RAPTOR | ★★★ (global ★★★★) | ★★★ | ★★★ | ★★★ | $$$ idx / $$ | ★★ | optional | niche | 85% |
| GraphRAG (lazy/light) | ★★★ (multi-hop/global ★★★★★) | ★★★★ | ★★★★ | ★★★–★★★★ | $–$$$ idx / $$ | ★★★ | ✓ (lazy) | ✓✓ OSINT | 90% |
| Agentic RAG | n/a (router) | ★★★★★ compound | ★★★★ | ★★★★ | $$ / $$$$ | ★★★ (budget-bound) | Phase 5 | essential | 85% |
| Reflection/Verification | n/a (wrapper) | +5–15 pts | ★★★★★ | ★★ | $ / $$ | ★★★★ | Phase 5–6 | essential | 88% |
| Multimodal RAG | ★★★★★ on visual | ★★★★ | ★★★ | ★★★ | $$ / $$ | ★★★★ (w/ pooling) | ✓✓ | essential here | 85% |

### 8.2 Expected golden-set accuracy trajectory (estimates — calibrate to your eval)

| Phase | Factoid | Multi-doc | Multi-hop | Global/theme | Visual/audio | Unanswerable handled | Basis |
|---|---|---|---|---|---|---|---|
| 1 Basic RAG | 55–65% | 40–55% | 15–30% | <10% | ~0% | <20% | CONSENSUS baselines |
| 2 Hybrid+Rerank | 72–80% | 60–72% | 25–40% | <15% | ~0% | <30% | FACT-anchored (RRF, rerank, contextual retrieval) |
| 3 Multimodal | 72–80% | 60–72% | 25–40% | <15% | 60–75% | <30% | ViDoRe-class gaps; ASR WER lit. |
| 4 GraphRAG | 72–80% | 65–75% | 45–65% | 60–75% | 60–75% | <35% | GraphRAG/LazyGraphRAG evals |
| 5 Agentic+Verify | 78–85% | 72–82% | 60–78% | 65–80% | 65–80% | >80% | Self-RAG/CRAG/CoVe-anchored, transfer=INFERENCE |
| 6 Reflection | +3–8 pts broadly; slope > step | | | | | >85% | Adaptive-RAG + grader-FT results |

*All Phase ≥3 numbers are INFERENCE from published benchmarks onto an unseen corpus; treat as planning priors, replace with measured values from Phase 1's harness.*

### 8.3 Best POC vs best production

| | Best POC architecture | Best production architecture |
|---|---|---|
| Retrieval | OpenSearch hybrid (BM25+ANN, RRF) + BGE-M3; ColQwen2.5 for doc images | Same pattern, scaled: OpenSearch cluster + Milvus; Vespa or Qdrant-multivector for visual; ColBERT-style hot-subset option |
| Precision | BGE-reranker-v2 / Jina v3 | Same + fine-tuned SLM grader; learning-to-rank with usage signals (Glean-style) |
| Structure | Neo4j single node; LightRAG/LazyGraphRAG | Neo4j/NebulaGraph cluster; ER pipeline; event/timeline model; incremental graph updates |
| Generation | Qwen3 7–32B via vLLM + Qwen2.5-VL | Tiered fleet (SLM/LLM/VLM) + optional frontier tier; quantized SLM roles |
| Orchestration | LangGraph behind own API; bounded loops | Same, hardened: budgets, queues (Kafka), K8s autoscaling, multi-tenant |
| Verification | CRAG grader + CoVe + citation enforcement | Full claim-level NLI + consensus + source-credibility store + abstention SLAs |
| Eval | RAGAS/DeepEval + golden sets + Langfuse | Continuous eval in CI; post-mortem loop; drift monitors; human annotation queue |

### 8.4 Recommended POC technology stack (one line each)

Parsing: **Docling (+PyMuPDF fast path)** · OCR: **Surya/Qwen-VL** · ASR: **WhisperX large-v3 + pyannote** · Text embed: **BGE-M3** · Visual retrieval: **ColQwen2.5** · Vector DB: **Qdrant** · Search: **OpenSearch** · Graph: **Neo4j + LightRAG/LazyGraphRAG** · Reranker: **BGE-reranker-v2-m3 / Jina v3** · LLM fleet: **Qwen3 (1.7B→32B) + Qwen2.5-VL via vLLM** · Agents: **LangGraph (wrapped)** · Eval: **RAGAS + DeepEval + Langfuse** · Pipelines: **Ray (batch) + Kafka-ready ingestion workers** · Deploy: **Docker → K8s**.

---

## 9. Trade-offs, Bottlenecks, and Closing Guidance

**Principal trade-offs.**
1. **Index-time vs query-time intelligence.** Full GraphRAG/RAPTOR front-load cost and go stale; LazyGraphRAG/agentic search pay at query time. OSINT corpora update constantly → bias toward query-time intelligence + cheap incremental indexes [CONSENSUS + FACT on lazy economics].
2. **Recall vs precision vs latency.** The funnel resolves it, but every added stage (grader, verifier, agent loop) trades latency for trust. Ship two modes: *fast* (Phase 2/3 path, seconds) and *investigate* (Phase 5 path, minutes) — exactly the Perplexity/Deep-Research product split [INFERENCE].
3. **Open-weight vs frontier models.** Sensitive OSINT pushes self-hosting; hardest synthesis quality pushes frontier APIs. The tiered-fleet design lets policy decide per corpus, not per architecture.
4. **Text-pipeline reuse vs visual-native fidelity.** Caption-everything is cheap and uniform; ColQwen is heavier but wins precisely on the scanned/visual material OSINT cares about. Run both on your scan slice in Phase 3b and let the eval choose per media type.

**Expected bottlenecks (in order of likely pain).**
1. **Parsing/ETL quality** — the silent killer; most "RAG failures" are ingestion failures [CONSENSUS]. Budget real time for the corpus census.
2. **Entity resolution** — graph value collapses with drifting entities; treat ER as a product, not a script.
3. **Evaluation labor** — golden sets and post-mortems are unglamorous and decisive; this is what Glean's relevance engineering actually is [FACT/INFERENCE].
4. **GPU budget for visual indexing + reranking** — plan pooling/quantization early.
5. **Agent cost control** — without hard budgets, investigate-mode costs are unbounded.
6. **VLM numeric hallucination** on charts/tables — needs its own eval + OCR cross-check.

**What would change these recommendations.** If your corpus turns out to be (a) overwhelmingly clean born-digital text → deprioritize Phase 3b; (b) dominated by one language with heavy slang/codewords → move embedding/reranker fine-tuning earlier; (c) mostly needed for point lookups, not investigations → stop at Phase 3 + thin verification and bank the savings.

**The single most important instruction in this document:** build the golden set and evaluation harness in week 1, and let measured deltas on *your* data — not these literature-anchored priors — gate every phase.
