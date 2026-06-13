# PRD: Multimodal OSINT Intelligence Chatbot — Proof of Concept

| Field | Value |
|---|---|
| Document | Product Requirements Document (POC) |
| Version | 1.1 — June 2026 (open-source / Ollama mandate) |
| Status | Draft for engineering review |
| Owner | Product / AI Architecture |
| Companion docs | (1) *Designing a Production-Grade Multi-TB OSINT Intelligence Chatbot* (end-state research); (2) *Incremental POC Plan* (architecture comparison & rationale) |
| Target duration | ~26 weeks, 6 gated phases |
| Team assumption | 2–3 engineers (1 backend/infra, 1 ML/retrieval, 0.5–1 full-stack), part-time analyst as domain SME |

> **HARD CONSTRAINT — FULLY OPEN-SOURCE, SELF-HOSTED, ZERO PAID SERVICES (v1.1).**
> Every model and every piece of infrastructure in this system MUST be open-weight / open-source and run on infrastructure we control. **No paid APIs, no hosted model endpoints, no commercial SaaS in the data or inference path** — not for generation, embeddings, reranking, OCR, ASR, parsing, or evaluation. The previous `external_models_allowed` escape hatch is **removed**.
> **Serving rule:** all *generative* models (LLMs, SLMs, VLMs) and *text-embedding* models are served through **Ollama**. Specialist non-generative models that Ollama does not serve — cross-encoder rerankers, ColQwen/ColPali visual late-interaction retrievers, Whisper ASR, speaker diarization, OCR, NER/relation extractors — remain fully open-source and self-hosted via their own open runtimes (HF Transformers / official repos). This split is a technical necessity, not a policy exception; see §6.5 and the changelog at the end.

---

## 1. Overview

### 1.1 Problem statement
Analysts possess a large, already-acquired multimodal corpus (HTML pages, PDFs — both born-digital and scanned — plain text, images, screenshots, video, audio) and currently cannot interrogate it efficiently. Keyword search misses semantics; manual review doesn't scale; existing chat-with-docs tools hallucinate, ignore non-text media, and cannot answer relationship or "big picture" questions. Wrong-but-confident answers are unacceptable in intelligence work.

### 1.2 Product vision (POC scope)
A chatbot that answers analyst questions **grounded exclusively in the provided corpus**, with:
- per-sentence citations down to page region / timestamp,
- explicit confidence and abstention ("insufficient evidence") behavior,
- coverage of all media types in the corpus,
- entity-relationship and corpus-wide thematic answering,
- two interaction modes: **Fast** (seconds) and **Investigate** (minutes, multi-step, verified).

### 1.3 What this PRD is
A build specification for developers: numbered requirements with acceptance criteria, interface contracts, data models, phase gates, and test plans. Architecture *rationale* (why hybrid+rerank, why LazyGraphRAG over full GraphRAG, why ColQwen for scans) lives in the companion POC Plan and is not re-argued here.

### 1.4 Explicit non-goals (POC)
- **NG-1** Data acquisition, crawling, or scraping — corpus is given.
- **NG-2** Face recognition, person re-identification, biometric matching.
- **NG-3** Real-time/streaming ingestion (batch + manual re-index only; design must not preclude it).
- **NG-4** Multi-tenant auth/RBAC beyond a single shared analyst login.
- **NG-5** Fine-tuning the main generator LLM.
- **NG-6** Production SLAs, HA, disaster recovery.
- **NG-7** Mobile clients; desktop web only.
- **NG-8** Automated actions/exports to external systems (read-only assistant).

---

## 2. Users & Personas

| Persona | Needs | Representative queries |
|---|---|---|
| **P1 Analyst** (primary) | Trustworthy answers with verifiable citations; entity profiles; timelines; relationship paths; ability to drill from claim → source document/page/timestamp | "What do we know about [person X]?" · "How are org A and org B connected?" · "Timeline of events involving X in 2021–22" · "Summarize the dominant narratives in this leak" · "What does the chart on page 4 of doc Y show?" |
| **P2 Reviewer/Lead** | Audit why an answer was given; see evidence quality, dissenting sources, confidence | Inspect answer trace, claim table, abstention reasons |
| **P3 Developer/ML engineer** | Debuggability: traces per stage, eval dashboards, reproducible indexes | Compare retrieval ablations; replay a failed query |

---

## 3. Success Metrics & Definitions

### 3.1 Golden evaluation set (the contract)
All quality requirements are measured on a versioned **golden set** (built in Phase 1, grown each phase):

| Query class | Code | Phase-1 count | Final count |
|---|---|---|---|
| Single-doc factoid | QF | 40 | 60 |
| Multi-document synthesis | QM | 25 | 40 |
| Multi-hop / relationship | QH | 15 | 35 |
| Global / thematic | QG | 10 | 30 |
| Visual (chart/scan/screenshot) | QV | 5 (will fail) | 30 |
| Audio/video | QA | 5 (will fail) | 20 |
| Unanswerable / false-premise / conflicting-source | QU | 10 | 35 |

Each item: question, query class, graded relevant evidence (doc/chunk/page/timestamp IDs), reference answer or rubric, "answerable: yes/no". Stored in repo as JSONL; eval runs in CI.

### 3.2 Metric definitions
- **Recall@k / nDCG@10**: against graded evidence per query.
- **Answer accuracy**: LLM-judge + human spot-check (≥20% sample/phase) against rubric; 3-point scale {correct, partially correct, wrong}; "accuracy" = % correct.
- **Citation precision**: % of cited passages that actually support the attached sentence (NLI/judge + human calibration).
- **Unsupported-claim rate**: % of answer sentences with no supporting citation that are not tagged `[ANALYSIS]`.
- **Abstention quality**: on QU set — % correctly abstained or premise-corrected.
- **Latency**: p50/p95 per mode. **Cost**: tokens + GPU-seconds per query, per mode.

### 3.3 Phase exit targets (gates)

| Gate | Must hit (on golden set, held-out split) |
|---|---|
| G1 (Phase 1) | Pipeline E2E works; baseline scorecard published; golden set v1 frozen |
| G2 | Recall@5 ≥ 0.70 overall; QF accuracy ≥ 72%; citation precision ≥ 80%; Fast-mode p95 ≤ 6 s |
| G3 | QV accuracy ≥ 60%; QA accuracy ≥ 55%; no regression >2 pts on text classes; media citations resolve to page/timestamp |
| G4 | QH accuracy ≥ 50%; QG acceptable-answer ≥ 60%; router accuracy ≥ 85% on labeled routing set; entity-merge precision ≥ 90% on labeled ER sample |
| G5 | QU handled ≥ 80%; unsupported-claim rate ≤ 5% (Investigate mode); QH ≥ 65%; Investigate p95 ≤ 4 min; cost/query within budget table §10 |
| G6 | Continuous-eval in CI gating deploys; 3 post-mortem cycles completed; fine-tuned grader ablation report; overall accuracy +3 pts vs G5 OR documented saturation analysis |

A phase does not close until its gate passes on the held-out split. Targets are planning priors; PM + tech lead may re-baseline once Phase-1 measurements exist, with written justification.

---

## 4. Functional Requirements

Numbering: `FR-<area>-<n>`. Priority: **M**ust / **S**hould / **C**ould (per phase noted).

### 4.1 Ingestion & Processing (ING)

- **FR-ING-1 (M, P1)** Batch ingestion CLI/API: point at a directory or manifest; system detects MIME type, routes to the correct parser, and is **idempotent** (re-runs skip unchanged files via content hash).
- **FR-ING-2 (M, P1)** Every source file gets a stable `doc_id` (content hash) and a metadata record: path, MIME, size, detected language(s), created/modified time, source collection tag, ingestion timestamp, parser used, parse-quality score.
- **FR-ING-3 (M, P1)** HTML → main-content extraction (boilerplate removed) preserving headings, lists, tables, link targets.
- **FR-ING-4 (M, P1)** Born-digital PDF/Office → structured text with headings, tables (as both markdown and cell-structure), reading order, page numbers retained per chunk.
- **FR-ING-5 (M, P1)** Chunking: recursive, ~512 tokens, 10–15% overlap, never split mid-table-row or mid-sentence; each chunk stores `doc_id, chunk_id, page/section anchor, char offsets`.
- **FR-ING-6 (M, P2)** Contextual chunk headers: prepend an LLM/SLM-generated 1–2 sentence document-context string to each chunk **before embedding and BM25 indexing** (Anthropic contextual-retrieval pattern). Header is stored separately so raw text can be displayed clean.
- **FR-ING-7 (M, P3)** Scanned PDFs/images of documents → OCR path (Surya/PaddleOCR) **and** page-render path (300 DPI PNG per page) for the visual index. OCR confidence stored per block; blocks <70% confidence flagged.
- **FR-ING-8 (M, P3)** Audio (and video audio tracks) → ASR with word-level timestamps + speaker diarization; transcript chunks anchored `[t_start, t_end, speaker]`; language auto-detected; ASR confidence stored; <threshold segments flagged.
- **FR-ING-9 (M, P3)** Video → keyframe extraction (scene-change based, max 1 frame/5 s) → VLM caption per keyframe with timestamp; captions indexed as text chunks; frames stored for display.
- **FR-ING-10 (M, P3)** Standalone images/screenshots → VLM caption + any VLM-OCR'd text + EXIF metadata → text index; image also embedded in image-embedding index.
- **FR-ING-11 (M, P3)** Visual document index: page renders embedded with ColQwen2.5-class model into multivector index; pooled single-vector also stored for coarse stage.
- **FR-ING-12 (M, P4)** Entity & relation extraction over text chunks using a **pinned, version-controlled extraction prompt/model**; output schema: `(entity{name, type∈{person,org,location,event,identifier,other}, aliases}, relation{src, type, dst, confidence}, provenance{chunk_ids})`.
- **FR-ING-13 (M, P4)** Entity resolution: deterministic keys (email/phone/ID/handle exact match) + embedding-similarity + alias rules → canonical entity IDs; every merge recorded and reversible; merge audit report generable.
- **FR-ING-14 (M, P4)** Knowledge graph persisted (Neo4j): entities, relations, event nodes (time, place, participants); **every node/edge carries provenance** → list of `chunk_id`s.
- **FR-ING-15 (S, P1)** Parse-quality sampling job: random 1% of parsed docs rendered side-by-side (original vs extracted) for human review; quality score logged.
- **FR-ING-16 (S, P4)** Incremental ingestion: adding new files updates text/vector/visual indexes and appends to graph without full rebuild (full graph re-cluster allowed weekly).

### 4.2 Retrieval (RET)

- **FR-RET-1 (M, P1)** Dense ANN search over chunk embeddings, top-k configurable, metadata filters (collection, date range, language, media type).
- **FR-RET-2 (M, P2)** BM25 search over chunk text (+ contextual headers) with the same filter set.
- **FR-RET-3 (M, P2)** Hybrid fusion: RRF over BM25 top-300 ∪ dense top-300 → 150 candidates; fusion parameters config-driven.
- **FR-RET-4 (M, P2)** Cross-encoder reranking of fused candidates → top-20; reranker model configurable; scores persisted in trace.
- **FR-RET-5 (M, P2)** Query preprocessing: SLM rewrite (spelling, expansion), alias expansion from alias table; original + rewritten both searched; rewrite shown in trace.
- **FR-RET-6 (M, P3)** Visual retrieval: query → ColQwen query embedding → coarse pooled search top-100 → late-interaction MaxSim rerank → top-10 pages; fused with text candidates via RRF before final rerank.
- **FR-RET-7 (M, P4)** Graph local search: entity linking on query → k-hop neighborhood (k≤3) → attached provenance chunks → join rerank pipeline.
- **FR-RET-8 (M, P4)** Graph global search: LazyGraphRAG-style — query expansion to subqueries, relevance-budgeted (configurable budget) iterative chunk testing + on-the-fly community/cluster summarization → map-reduce answer context.
- **FR-RET-9 (M, P4)** Query router: SLM classifier → {factoid | entity/multi-hop | global | visual | timeline | mixed}; routes to retrieval strategy; low-confidence default = hybrid text; router decision + confidence in trace.
- **FR-RET-10 (M, P1)** **Retrieval API is a versioned internal service**: `POST /retrieve {query, mode, filters, k, strategy?}` → ranked evidence list with scores per stage. All later layers (agents, UI, eval) consume only this API.
- **FR-RET-11 (S, P4)** Timeline retrieval: entity/topic → event nodes sorted by time with evidence; returned as structured timeline objects.

### 4.3 Generation & Answering (GEN)

- **FR-GEN-1 (M, P1)** Answer composed **only** from retrieved evidence; system prompt forbids external knowledge for factual claims.
- **FR-GEN-2 (M, P1)** Inline citations: every factual sentence carries ≥1 citation marker resolvable to `chunk_id` (and page/timestamp where applicable). Sentences that are the model's synthesis/judgment must be tagged `[ANALYSIS]`.
- **FR-GEN-3 (M, P1)** If evidence is insufficient: explicit "insufficient evidence in corpus" answer; never fill gaps from parametric knowledge.
- **FR-GEN-4 (M, P3)** Multimodal answering: when visual pages retrieved, VLM receives page images; citations reference `doc_id+page` (+bounding region where available); audio citations reference `doc_id+[t_start–t_end]`.
- **FR-GEN-5 (M, P3)** Chart/number safeguard: numeric claims read from images are cross-checked against OCR tokens of the same region; disagreement → claim flagged `[LOW CONFIDENCE — visual read]`.
- **FR-GEN-6 (M, P2)** Context assembly: dedup near-identical chunks, group by source, prepend source headers (title, date, type, credibility tag), hard token budget.
- **FR-GEN-7 (M, P4)** Graph answers include human-readable path explanation ("A —director→ X —shareholder→ B") with per-edge citations.

### 4.4 Agentic Orchestration (AGT) — Phase 5

- **FR-AGT-1 (M)** Two modes: **Fast** (single-pass Phase-2/3/4 routed pipeline) and **Investigate** (planner loop). Mode user-selectable; system may suggest escalation.
- **FR-AGT-2 (M)** Planner: decomposes compound queries into sub-questions, selects retrieval strategy per sub-question, declares a plan object (shown to user, streamed).
- **FR-AGT-3 (M)** Bounded execution: hard caps — max agent iterations (default 8), max total tokens, max wall-clock (default 4 min), max retrieval calls; on cap hit, synthesize best-effort answer with explicit "budget exhausted" note.
- **FR-AGT-4 (M)** CRAG-style retrieval grading: SLM grades each sub-question's evidence {sufficient | ambiguous | insufficient}; on non-sufficient → query rewrite, filter broadening, or alternate strategy (≤2 retries per sub-question).
- **FR-AGT-5 (M)** Full trace persisted: plan, every tool call, grader verdicts, budgets consumed; viewable in UI (P2/P3 personas).

### 4.5 Verification & Confidence (VER) — Phase 5–6

- **FR-VER-1 (M, P5)** Claim extraction: final draft split into atomic claims; each mapped to its citations.
- **FR-VER-2 (M, P5)** Claim-level grounding check (NLI/judge model ≠ generator): {supported | partial | unsupported}; unsupported claims removed or tagged; results in answer's claim table.
- **FR-VER-3 (M, P5)** Chain-of-verification for Investigate mode: verification questions generated and answered against evidence before final revision.
- **FR-VER-4 (M, P5)** Per-claim confidence score = f(#independent sources, source agreement, source credibility, retrieval rank, grounding verdict); displayed as High/Medium/Low with tooltip breakdown. Sensitive-claim rule: single-source claims always flagged.
- **FR-VER-5 (M, P5)** Conflicting evidence is surfaced, not averaged: "Source A states X (2021); Source B states Y (2023)."
- **FR-VER-6 (S, P5)** Source credibility store: per-source reliability tag (seed: analyst-assigned tiers; updatable), feeding FR-VER-4.
- **FR-VER-7 (M, P6)** Adaptive retrieval: complexity classifier may skip retrieval (chitchat/meta), single-shot, or iterate; decisions logged.
- **FR-VER-8 (M, P6)** Reflective critic pass on each answer segment (supported? useful?) with regenerate/trim loop, max 1 revision in Fast, 2 in Investigate.

### 4.6 User Interface (UI)

- **FR-UI-1 (M, P1)** Chat interface: streaming answers, markdown rendering, citation chips inline.
- **FR-UI-2 (M, P1)** Citation drill-down: click chip → side panel with source passage highlighted in document context; for media: page image with region highlight / audio player seeked to timestamp / video frame.
- **FR-UI-3 (M, P2)** Filters panel: collection, date range, media type, language — applied to retrieval.
- **FR-UI-4 (M, P4)** Entity page: canonical entity → aliases, attributes, relation list, mini-graph view, timeline, mention list with links.
- **FR-UI-5 (M, P4)** Graph view: interactive neighborhood explorer (expand node, filter edge types); answer-path highlighting.
- **FR-UI-6 (M, P5)** Investigate mode UX: streamed plan → progress per sub-question → final answer + claim table (claim, confidence, sources, verdict) + dissent section.
- **FR-UI-7 (M, P1)** Feedback: thumbs up/down + free-text per answer; stored with full trace for post-mortems.
- **FR-UI-8 (S, P5)** Trace viewer (developer/reviewer): all stages, scores, budgets, model calls.

### 4.7 Evaluation & Ops Tooling (EVAL)

- **FR-EVAL-1 (M, P1)** Golden-set harness: run any pipeline config against golden set; outputs scorecard (all §3.2 metrics) as artifact; runnable locally and in CI.
- **FR-EVAL-2 (M, P1)** Tracing on every request (OpenTelemetry-compatible → Langfuse/Phoenix): per-stage inputs, outputs, scores, latency, token/GPU cost.
- **FR-EVAL-3 (M, P2)** Ablation mode: run golden set across config matrix (dense-only / bm25-only / hybrid / +rerank, etc.) and diff scorecards.
- **FR-EVAL-4 (M, P6)** CI gate: PRs touching retrieval/generation configs run golden-set eval; regression >2 pts on any class blocks merge.
- **FR-EVAL-5 (M, P6)** Post-mortem queue: low-confidence + thumbs-down traces land in a triage queue; resolution workflow adds labeled items to golden set.
- **FR-EVAL-6 (S, P6)** Drift dashboard: retrieval score distributions, abstention rate, per-class accuracy over time.

---

## 5. Non-Functional Requirements

| ID | Requirement | Target (POC) |
|---|---|---|
| NFR-1 Latency | Fast mode p50 / p95 | ≤ 3 s / ≤ 6 s |
| NFR-2 Latency | Investigate mode p95 | ≤ 4 min (hard cap, FR-AGT-3) |
| NFR-3 Throughput | Concurrent users | 5 concurrent analysts without degradation |
| NFR-4 Ingestion | Throughput on POC slice | Full corpus slice (1–5%, stratified) ingestible in ≤ 48 h on dev hardware |
| NFR-5 Data locality & model sovereignty | All corpus data, derived artifacts, **and all model inference** remain on infrastructure we control. **No corpus content and no query content may leave the boundary**; no calls to any external/hosted model or SaaS in the data or inference path. All models are open-weight and run locally (Ollama for generative + text-embeddings; open runtimes for specialist models). This is unconditional — there is no clearance flag. |
| NFR-6 Reproducibility | Indexes rebuildable from raw corpus + pinned configs (model versions, prompts, chunking params all version-controlled); embedding/model versions stamped on every record |
| NFR-7 Auditability | Every answer reconstructable from its trace ID for ≥ 90 days |
| NFR-8 Cost | Per-query cost logged; budgets per §10; monthly POC infra within agreed envelope |
| NFR-9 Security | Single-tenant; TLS; corpus storage encrypted at rest; secrets in vault; no PII leaves the boundary |
| NFR-10 Resilience | Any single service restart must not corrupt indexes; ingestion resumable from checkpoint |
| NFR-11 Observability | All services emit structured logs + metrics (Prometheus) + traces; one Grafana board per phase gate metric |
| NFR-12 Licensing | **Every** model and component MUST be open-weight/open-source under a license permitting internal commercial use (e.g. Apache-2.0, MIT, Llama/Qwen community, CC-BY). No proprietary or usage-metered components anywhere. License + weight-source inventory maintained and reviewed at each gate. |

**Ethical/legal guardrails (POC-level, M):**
- **NFR-13** The system answers about the corpus; it must refuse to fabricate intelligence beyond evidence (enforced by FR-GEN-1/3, FR-VER-2).
- **NFR-14** No biometric identification features (NG-2); no automated targeting recommendations; outputs are decision support with human-in-the-loop framing.
- **NFR-15** Source material may itself be false/propaganda: credibility tags (FR-VER-6) and conflict surfacing (FR-VER-5) are mandatory, not cosmetic.

---

## 6. System Architecture & Interfaces

### 6.1 Service decomposition (target by Phase 5)

```
                                ┌─────────────┐
  Web UI (React) ──────────────►│ API Gateway │
                                └──────┬──────┘
                 ┌─────────────────────┼───────────────────────┐
                 ▼                     ▼                       ▼
          ┌────────────┐       ┌──────────────┐        ┌──────────────┐
          │ Chat/Agent │──────►│ Retrieval API│        │  Eval Service│
          │ Orchestr.  │       │  (router +   │        │ (golden set, │
          │ (LangGraph │       │  strategies) │        │  ablations)  │
          │  wrapped)  │       └──┬───┬───┬───┘        └──────────────┘
          └─────┬──────┘          │   │   │
                │            ┌────┘   │   └─────────┐
                ▼            ▼        ▼             ▼
        ┌─────────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
        │Model Gateway│ │OpenSearch│ │ Qdrant  │ │  Neo4j  │
        │ Ollama:LLM/ │ │BM25+filt.│ │dense +  │ │ KG +    │
        │ SLM/VLM/emb │ └─────────┘ │multivec  │ │ GDS     │
        │ +open rt:   │             │(ColQwen) │ └─────────┘
        │ rerank/ASR/ │             └─────────┘
        │ OCR/NER/Col │
        └─────────────┘
                ▲
        ┌───────┴────────┐        ┌──────────────────┐
        │ Verification   │        │ Ingestion workers │──► object store (raw,
        │ Service (NLI,  │        │ (parse/OCR/ASR/   │     renders, frames,
        │ CoVe, scoring) │        │  caption/extract) │     audio segments)
        └────────────────┘        └──────────────────┘
```
Phases 1–2 may run as a modular monolith, but **module boundaries must match these service boundaries** so extraction to services is mechanical.

### 6.2 Core data model (canonical records)

```jsonc
// Document
{ "doc_id": "sha256:…", "collection": "leakset-A", "mime": "application/pdf",
  "title": "...", "lang": ["en","ru"], "source_path": "...", "created": "...",
  "credibility_tier": "B", "parse": {"parser":"docling@2.x","quality":0.93} }

// Chunk (text & transcript & caption all share this)
{ "chunk_id": "doc:…#c0042", "doc_id": "...", "text": "...", "context_header": "...",
  "anchor": {"page": 7} | {"t_start": 312.4, "t_end": 339.0, "speaker":"S2"} | {"frame_ts": 84.0},
  "char_span": [10412, 12047], "lang": "en",
  "embeddings": {"bge-m3@1": "ref"}, "ocr_conf": 0.88, "media_type": "pdf_scan" }

// VisualPage
{ "page_id": "doc:…#p7", "doc_id": "...", "render_uri": "s3://…/p7.png",
  "colqwen_multivec_ref": "...", "pooled_vec_ref": "...", "ocr_tokens_ref": "..." }

// Entity / Relation (graph)
{ "entity_id": "ent:…", "type": "person", "canonical_name": "...",
  "aliases": ["..."], "merged_from": ["ent:…"], "provenance": ["chunk_id", …] }
{ "rel_id": "rel:…", "src": "ent:a", "type": "DIRECTOR_OF", "dst": "ent:b",
  "confidence": 0.81, "provenance": ["chunk_id", …], "extractor": "qwen3-4b@prompt-v3" }

// AnswerTrace
{ "trace_id": "...", "mode": "investigate", "plan": {...}, "stages": [...],
  "claims": [{"text":"…","citations":["chunk_id"],"verdict":"supported","confidence":0.9}],
  "budgets": {"iters": 5, "tokens": 41200, "wall_s": 96}, "feedback": null }
```

### 6.3 Key API contracts (internal, versioned)

```
POST /v1/retrieve
  { query, strategy?: auto|hybrid|graph_local|graph_global|visual|timeline,
    filters?: {collections[], date_range, media_types[], langs[]},
    k?: int, trace_id }
→ { candidates: [{chunk_id|page_id, scores:{bm25,dense,rrf,rerank,maxsim?},
    anchor, snippet}], router: {label, confidence}, timings }

POST /v1/answer        // Fast mode
  { query, filters?, trace_id } → SSE stream: tokens + citation events + final claim table

POST /v1/investigate   // Phase 5
  { query, filters?, budgets?: {...} } → SSE: plan → subq progress → draft → verified answer

POST /v1/ingest  { manifest_uri, collection } → job_id ;  GET /v1/ingest/{job_id}
POST /v1/eval/run { config_ref, golden_set_version } → scorecard artifact
GET  /v1/entities/{id} ; GET /v1/graph/neighborhood?entity=…&hops=…
```

### 6.4 Configuration as code
Single `pipeline.yaml` per environment: model names+versions, prompts (referenced by file+hash), chunking params, fusion weights, k values, budgets, feature flags (`visual_index`, `graph`, `verification`). Every trace records the config hash it ran under. **Acceptance: two runs with the same config hash on the same index are reproducible (modulo sampling temperature, which is 0 for graders/routers).**

### 6.5 Open-source model stack & serving (MANDATORY)

All models below are open-weight and self-hosted. Two runtimes only:
- **Ollama** — serves all generative models (LLM/SLM/VLM) and text-embedding models that have GGUF/Ollama support. One process, one model registry, OpenAI-compatible local endpoint.
- **Open specialist runtimes** — models Ollama does not serve, run via HF Transformers / their official repos in our own GPU containers. Still open-source, still on-prem, still zero paid APIs.

| Role | Model (open-weight) | Served by | Notes / why |
|---|---|---|---|
| Synthesis / planner (Investigate) | **Qwen3-32B** (or Llama-3.3-70B if hardware allows), quantized | Ollama | Strongest open synthesis we can self-host; INT4/Q4 quant |
| Workhorse generation (Fast) | **Qwen3-8B** | Ollama | Main generator/reasoner |
| SLM roles: router, CRAG grader, query-rewrite, citation-aligner | **Qwen3-1.7B / Qwen3-4B** | Ollama | Cheap, fast, temp=0 |
| VLM: answering over page images, captioning, VLM-OCR fallback | **Qwen2.5-VL-7B** | Ollama (vision) | Multimodal answering + caption/OCR |
| Text embeddings | **BGE-M3** | Ollama (`bge-m3`) | Dense + multilingual; available as Ollama embedding model |
| Verification / NLI claim-check | **Qwen3-4B** as judge (prompted) + open NLI checker | Ollama (+ open rt) | Judge model MUST differ from generator (FR-VER-2) |
| Cross-encoder reranker | **BGE-reranker-v2-m3** (or **Jina-reranker-v2**, open-weight) | open runtime | Not an Ollama model type; runs in own container |
| Visual late-interaction retriever | **ColQwen2.5** (or ColModernVBERT for speed) | open runtime | Multivector/MaxSim; not Ollama-servable |
| ASR | **Whisper large-v3** via faster-whisper / WhisperX | open runtime | Word timestamps + diarization |
| Speaker diarization | **pyannote** (open) | open runtime | Open weights (gated download, free) |
| OCR | **Surya** / **PaddleOCR** | open runtime | Bulk OCR; VLM-OCR fallback via Qwen2.5-VL |
| NER / relation extraction | **GLiNER** + Qwen3-4B extractor | open rt + Ollama | Pinned extraction prompt (FR-ING-12) |
| Entity-resolution embeddings | BGE-M3 reused | Ollama | — |

**Engineering note (throughput trade-off):** Ollama optimizes for ease of operation and single-node simplicity, not maximum batched throughput — at production scale a batching server (e.g. vLLM/TGI, both open-source) would serve the same open weights faster. **For the POC, Ollama is the mandated serving layer** (NFR-3 only requires 5 concurrent users, which Ollama handles). The Model Gateway exposes an OpenAI-compatible interface so the serving backend can later be swapped to vLLM/TGI **with zero application-code change** — this is recorded as a deferred production option, not a POC task.

**`pipeline.yaml` must pin:** Ollama model tags + digests, specialist model repo + commit hash, quantization level, and prompt file hashes. License inventory (NFR-12) lists each weight's source and license.

---

## 7. Phase Plan: Scope, Deliverables, Acceptance

(Requirements mapped per phase; gates in §3.3. Durations indicative for the §0 team.)

### Phase 1 — Foundation & Basic RAG (Wks 1–3)
**Build:** FR-ING-1…5, 15; FR-RET-1, 10; FR-GEN-1…3, 6(basic); FR-UI-1, 2(text), 7; FR-EVAL-1, 2. Corpus census job (format/language/quality distribution report). Golden set v1 (§3.1) authored with analyst SME.
**Definition of done:** G1 + demo: analyst asks 10 live questions, every answer's citations open the right passage; scorecard artifact committed.

### Phase 2 — Hybrid + Rerank (Wks 3–6)
**Build:** FR-ING-6; FR-RET-2…5; FR-GEN-6(full); FR-UI-3; FR-EVAL-3. Retrieval API hardened (versioned, documented).
**DoD:** G2 + ablation report (≥4 configs) committed; latency dashboard live.

### Phase 3 — Multimodal (Wks 6–11)
**Build:** FR-ING-7…11; FR-RET-6; FR-GEN-4, 5; FR-UI-2 (media drill-down: page region, audio seek). Golden set v2 (+QV/QA). Chart-QA mini eval. Bake-off report: OCR-text vs ColQwen vs fused on scanned slice → per-media-type routing decision recorded as ADR.
**DoD:** G3 + media citations verified by analyst on 20 sampled answers.

### Phase 4 — Graph (Wks 11–16)
**Build:** FR-ING-12…14, 16; FR-RET-7…9, 11; FR-GEN-7; FR-UI-4, 5. Golden set v3 (+QH/QG/timeline). ER evaluation on labeled entity sample. Router labeled set (≥200 queries) + router eval.
**DoD:** G4 + entity page demo on 5 analyst-chosen entities; extraction prompt + model pinned with version tag; merge audit report.

### Phase 5 — Agentic + Verification (Wks 16–21)
**Build:** FR-AGT-1…5; FR-VER-1…6; FR-UI-6, 8. Red-team set (50 adversarial queries) authored and run. Cost meter per mode.
**DoD:** G5 + reviewer persona walkthrough: pick 5 answers, audit each fully from claim table to sources without developer help.

### Phase 6 — Reflection & Continuous Eval (Wks 21–26)
**Build:** FR-VER-7, 8; FR-EVAL-4…6. Three post-mortem cycles executed. Optional: LoRA fine-tune of relevance grader from accumulated traces, with before/after ablation. Final POC report: metric trajectory P1→P6, cost/query, production scaling plan referencing companion architecture doc.
**DoD:** G6 + handover: a new engineer can rebuild all indexes from raw corpus using docs alone (tested).

---

## 8. Test Plan Summary

| Layer | Tests |
|---|---|
| Parsing | Per-format fixture suite (≥10 fixtures/format incl. adversarial: rotated scans, multi-column, RTL text, corrupt files); parse-quality sampling (FR-ING-15) |
| Chunking | Property tests: no mid-sentence/table-row splits; anchor integrity (chunk text == doc[char_span]) |
| Retrieval | Golden-set Recall/nDCG in CI; filter correctness unit tests; router confusion matrix on labeled set |
| ER/Graph | Labeled merge sample (precision/recall); provenance integrity check (every edge's chunks exist & mention both endpoints — sampled LLM audit) |
| Generation | Citation resolution test (every marker resolves); unsupported-claim scan; chart numeric cross-check suite |
| Agents | Budget-cap enforcement tests; loop/termination tests; deterministic replay from trace |
| Verification | NLI checker calibration vs 200 human-labeled claim-citation pairs (target ≥85% agreement) |
| Adversarial | QU set + red-team set: false premise, prompt injection **embedded in corpus documents** (indirect injection — system must not follow instructions found inside retrieved content; explicit test cases), conflicting sources, data-exfil prompts |
| Load | 5 concurrent users × 30 min soak, Fast mode, p95 within NFR-1 |

---

## 9. Infrastructure & Environments

| Item | POC provision |
|---|---|
| Compute | 1 GPU server (2× A100/H100-class or 4× L40S): **Ollama** serving LLM/SLM/VLM/embeddings + open-runtime containers for reranker, ColQwen, Whisper, OCR, NER + GPU slice for ColQwen indexing bursts; 1 CPU server (32–64 vCPU, 256 GB) for OpenSearch+Qdrant+Neo4j+services; object store (MinIO — open-source, S3-compatible) for raw + renders + frames. **All software self-hosted; no managed cloud services required.** |
| Environments | `dev` (engineers), `eval` (frozen for gate runs), shared single-node k8s or docker-compose; IaC from day 1 (even if compose) |
| Corpus slice | 1–5% stratified sample (by format × language × collection), selected with analyst; documented sampling script |
| Model registry | All weights mirrored locally; versions pinned in `pipeline.yaml` |
| Backups | Nightly: configs, golden sets, graph dump, metadata DB. Indexes are rebuildable (NFR-6), not backed up |

---

## 10. Cost & Budget Guardrails (self-hosted — compute only)

Because there are no paid APIs, "cost" is GPU/CPU time and latency, not per-token billing. Budgets are expressed as resource ceilings.

| Item | Guardrail |
|---|---|
| Fast-mode query | ≤ ~4 GPU-seconds total (retrieval + rerank + 1 generation pass); p95 ≤ 6 s (NFR-1) |
| Investigate query | Bounded by hard caps (FR-AGT-3): ≤ 8 iterations, ≤ 4 min wall-clock; resource ceiling enforced, not a dollar figure |
| Phase-3 visual indexing | ≤ 1 GPU-week for POC slice on Ollama/ColQwen; pooling/quantization mandatory if exceeded |
| Phase-4 extraction | Open SLM extractor (Qwen3-4B via Ollama) for all bulk extraction; no per-call billing to optimize against |
| Capital/infra | One-time hardware (or owned GPU server) + electricity; weekly utilization report from traces to size production from real numbers |

**Implication of the open-source mandate:** zero marginal API cost per query (the usual dominant RAG operating expense is eliminated), traded for upfront hardware capital and the engineering cost of self-hosting. This strengthens the data-sovereignty posture (NFR-5) and removes vendor lock-in, at the cost of carrying ops burden in-house.

---

## 11. Risks & Mitigations (top 10)

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| 1 | Parsing/ETL garbage poisons everything downstream | High | High | Corpus census + FR-ING-15 sampling + per-format fixtures before Phase-2 starts |
| 2 | Golden set too easy → fake green gates | Med | High | Analyst-authored hard cases; held-out split; rotate items; human spot-checks of judge |
| 3 | Entity drift wrecks graph value | High | Med-High | FR-ING-13 ER + labeled merge eval + reversible merges |
| 4 | VLM hallucinates chart/table numbers | High | High (trust) | FR-GEN-5 OCR cross-check + chart-QA eval + low-confidence flags |
| 5 | Agent cost/latency blowout | Med | Med | FR-AGT-3 hard budgets; cost meter; mode split |
| 6 | LLM-judge eval drifts from human judgment | Med | High | ≥20% human spot-check per gate; §8 calibration set |
| 7 | Indirect prompt injection via corpus documents | Med | High | §8 adversarial suite; instruction-hierarchy prompting; retrieved content rendered as data, never as instructions |
| 8 | Model/tool churn (frameworks, leaderboards) | High | Med | Own service boundaries (FR-RET-10); config-as-code; quarterly model review, not mid-phase swaps |
| 9 | ASR/OCR failure on noisy multilingual media | Med | Med | Confidence flags; language-ID routing; degraded-media report to stakeholders (set expectations) |
| 10 | Scope creep (face rec, live feeds, exports) | High | Med | Non-goals §1.4 enforced via change-control: any NG change requires PRD revision sign-off |

---

## 12. Open Questions (resolve by end of Phase 1)

1. **Corpus composition reality** — census will determine Phase-3 effort split (scan-heavy vs born-digital) and language list.
2. **Hardware sizing for the open-source fleet** — the synthesis model tier (Qwen3-32B vs Llama-3.3-70B) and concurrent visual indexing set the GPU floor. Confirm available GPU memory/count before Phase-3 planning; quantization level chosen accordingly. (Replaces the former external-model-policy question — that is now closed by the open-source mandate.)
3. **Analyst availability** — golden-set authoring and gate reviews need ~2 days/phase of SME time; named individual required.
4. **Credibility tier seed** — who supplies the initial per-source reliability taxonomy (FR-VER-6)?
5. **Hardware procurement** — confirm GPU server spec/lead time before Phase-3 planning.
6. **Languages in scope for ASR/OCR quality targets** — define the top-N languages the gates are measured on.
7. **Retention/handling rules** for derived artifacts (transcripts of sensitive audio, rendered pages) — same classification as source?

---

## 13. Glossary
**RRF** Reciprocal Rank Fusion · **Cross-encoder** joint query-passage relevance scorer · **Late interaction / MaxSim** token/patch-level similarity scoring (ColBERT/ColPali) · **CRAG** Corrective RAG retrieval grading · **CoVe** Chain-of-Verification · **LazyGraphRAG** query-time, budgeted graph summarization · **ER** entity resolution · **Golden set** versioned evaluation question set · **Gate** phase exit criteria (§3.3) · **Trace** full per-request execution record · **ADR** architecture decision record.

---
*End of PRD v1.0. Changes to goals, non-goals, or gates require sign-off from product owner + tech lead; all other sections are maintained by the engineering team via ADRs.*

---

## 14. Changelog — v1.0 → v1.1 (Open-Source / Ollama mandate)

| # | What changed | Where | Why |
|---|---|---|---|
| C1 | Added a top-of-document **HARD CONSTRAINT** banner: fully open-weight, self-hosted, zero paid services anywhere in the data/inference path | Header | Make the constraint impossible to miss and binding on all downstream choices |
| C2 | **Removed the `external_models_allowed` flag entirely** | NFR-5, §6.4 config flags, Open Question #2 | The escape hatch directly contradicts a no-paid-API mandate; data sovereignty is now unconditional |
| C3 | Rewrote **NFR-5** to forbid any external/hosted model or SaaS, for both corpus *and query* content | §5 | Closes the loophole where queries (not just corpus) might be sent out |
| C4 | Strengthened **NFR-12** to require open-weight/open-source licensing for *every* component, with a maintained weight-source + license inventory | §5 | Turns "preferred" into "required"; gives reviewers an auditable artifact |
| C5 | Replaced the **vLLM** serving layer with **Ollama** for all generative + text-embedding models | Architecture diagram, new §6.5, Compute §9 | Per your instruction. Ollama = single-process, simple ops, OpenAI-compatible local endpoint, sufficient for the POC's 5-concurrent-user target |
| C6 | Added **§6.5 Open-Source Model Stack & Serving** — a full role→model→runtime table (all open weights) and the explicit Ollama vs open-runtime split | New §6.5 | **Critical correction:** Ollama only serves LLMs/VLMs/text-embeddings. Rerankers, ColQwen, Whisper, diarization, OCR, NER are *not* Ollama-servable. They stay open-source but run on their own open runtimes. Saying "all via Ollama" would have made the PRD un-buildable |
| C7 | Pinned concrete open models: Qwen3-32B/8B/4B/1.7B, Qwen2.5-VL-7B, BGE-M3, BGE-reranker-v2-m3 / Jina-reranker-v2, ColQwen2.5, Whisper large-v3, pyannote, Surya/PaddleOCR, GLiNER | §6.5 | Removes the earlier "hosted Claude/Gemini/Cohere where policy allows" options; every named alternative is now open-weight |
| C8 | Added an **engineering note** that the OpenAI-compatible Model Gateway lets the backend be swapped from Ollama to vLLM/TGI (both open-source) at production scale with no app-code change | §6.5 | Honest throughput trade-off: Ollama favors simplicity over batched throughput. Keeping the abstraction means the POC choice doesn't trap the production system |
| C9 | Reframed **§10 Cost** from per-token dollar budgets to **self-hosted compute (GPU-seconds) budgets**, and noted zero marginal API cost per query | §10 | With no paid APIs, the dominant RAG operating expense disappears; budgets must be expressed as resource ceilings, not billing |
| C10 | Changed object store wording to **MinIO (open-source, S3-compatible)** and "no managed cloud services required" | §9 | Consistency with the self-hosted mandate; avoids implying a paid S3 dependency |
| C11 | Replaced Open Question "external model policy" with **"hardware sizing for the open-source fleet"** | §12 | The relevant unknown is now GPU capacity for self-hosted models, not API clearance |

**Net effect on the plan:** quality ceiling on the hardest synthesis questions is modestly lower than a frontier-API option would give (open 32–70B vs frontier models), and ops burden moves in-house. In exchange you get full data sovereignty, zero per-query cost, no vendor lock-in, and reproducible, air-gappable deployment — the right trade for sensitive OSINT. No functional requirements (FR-*) were deleted; only the models and serving layer behind them changed.
