# L2 Logic Checklist

Every piece of logic in the Mallory **Layer 2** engine — **what it's for** and **where it lives** —
organized by the path a crawled page actually travels: ingest → process → serve.

**The one axis that defines this codebase:** numbers & rankings are *always* deterministic;
the LLM only reads, judges, and writes prose.

**Tags**

| Tag | Meaning |
|-----|---------|
| **DET** | Deterministic — rules / math, never an LLM |
| **LLM** | LLM-first — judgment / prose, deterministic fallback |
| **HYB** | Hybrid — LLM proposes, deterministic gates & computes |
| **PLB** | Plumbing — boot, I/O, schema, transport |

**At a glance:** 43 modules · 13 pipeline stages · 7 LLM task verbs · **0 LLM-computed numbers** · 4 table namespaces

---

## 1. The processing pipeline — 13 stages (a real sequence)

One scheduler tick runs these in order over pending staging rows.
Orchestrated by `pipeline/runner.py → process_pending`. Each stage consumes the last stage's output.

| # | Tag | Logic | Goal | Location |
|---|-----|-------|------|----------|
| 01 | HYB | **Extraction** | Turn one bare crawled page into typed records (signal + optional tender/partnership/geo/event). LLM reads; on empty/invalid/offline it falls back to a regex extractor — so ingest never depends on a model. | `services/extraction.py → extract_pending` |
| 02 | DET | **Entity resolution** | Confirm/repair the crawler's competitor link against `ref_competitors` — exact id, then word-boundary alias match ("PEL" won't match inside "proPELlant"). An id is trusted only if it exists in reference data. | `services/entity_resolution.py → resolve_competitor` |
| 03 | HYB | **Signal pipeline** | Received signal → resolve → classify (LLM: threat/watch/fav) → enrich (LLM prose) → score confidence (deterministic) → publish card + detail + evidence. | `services/signal_pipeline.py → process_signal` |
| 04 | DET | **Domain pipeline** | Publish partnerships, geo footprints, innovations with templated "vs KSSL" prose (no LLM) and natural-key upsert (same fact from a 2nd source updates one card, not two). | `services/domain_pipeline.py → process_partnership / _geo / _innovation` |
| 05 | HYB | **Tender scoring** | Auto-score every KSSL product against a tender spec-by-spec → fit% with up/down reasons (deterministic), then ask the LLM for a go/maybe/pass verdict it may interpret but not renumber. | `services/tender_scoring.py → process_tender` |
| 06 | HYB | **Corroboration** | Count independent sources behind each claim — deterministic claim-key grouping, optionally merged by embedding similarity (Phase C). Feeds the confidence score. | `services/corroboration.py → corroboration_counts` |
| 07 | LLM | **Competitor synthesis** | The analyst brain: strategic read per rival (thesis, vulnerabilities, predictions) from its evidence pack. Uncited vulnerabilities are discarded; a thin/failed run never overwrites the seed row. | `services/competitor_synthesis.py → synthesize_all` |
| 08 | HYB | **Matchup synthesis** | Head-to-head KSSL-vs-rival edge score — per-spec leader by polarity, highlight specs weighted ×2 (deterministic). LLM writes only the verdict prose. | `services/matchup_synthesis.py → recompute_all` |
| 09 | HYB | **Field patterns** | Cross-field aggregates — shared-partner chokepoints, contested countries, licensing concentration (deterministic; only `sourced` rows count). LLM narrates; fallback publishes aggregates raw. | `services/field_patterns.py → refresh_field_patterns` |
| 10 | DET | **Graph build** | Project the relational system of record (ref_*/srv_*) into `kg_nodes / kg_edges` — full idempotent rebuild, every edge carrying provenance, confidence, evidence links. | `services/graph_builder.py → rebuild_graph` |
| 11 | DET | **Graph analytics** | Mine the graph (NetworkX, seeded): chokepoints, alliance blocs (Louvain), key brokers (betweenness), and predicted bidders — who'll show up against KSSL on a tender. | `services/graph_analytics.py → run_analytics` |
| 12 | DET | **Overview metrics** | Pre-compute the per-pillar metric strip so the client renders no math. Writes `srv_overview_metrics` (the row that 404s until this runs). | `services/metrics.py → build_overview_metrics` |
| 13 | DET | **Rank** | Order all published cards within each pillar — threats first, then recency. The rank the dashboard reads. | `services/signal_pipeline.py → recompute_ranks` |

---

## 2. Trust & grounding primitives (shared across the pipeline)

The three deterministic tools every card leans on. None ever touch an LLM.

| Tag | Logic | Goal | Location |
|-----|-------|------|----------|
| DET | **Confidence scoring** | The trust number: `source_tier(≤35) + corroboration(≤25) + freshness(≤25) + provenance(15)`, clamped 5–95, decomposed into visible parts. Rule-based forever. | `services/confidence.py → score` |
| DET | **Evidence ledger** | The grounding contract: stable evidence-ids (`sig:`/`geo:`/`agg:`…) + a write helper linking every serving field to the rows behind it — powers `/explain`. | `services/evidence.py → write_evidence` |
| DET | **Spec vocabulary** | Normalizes raw spec labels into comparable slots with unit, polarity, and operator (≥/≤/=) — the shared coordinate system for tender fit and matchup edge. | `services/spec_extract.py → parse_requirements` |

---

## 3. The LLM subsystem — the brain (7 files)

Everything the model can be asked, wrapped so its output is validated, grounded, cached,
and always falls back to a deterministic floor.

| Tag | Logic | Goal | Location |
|-----|-------|------|----------|
| PLB | **Provider protocol & factory** | Defines the 7 verbs L2 can ask a model (classify, enrich, verdict, chat, extract, caption, specs) and picks the provider from config (LLM_TARGET switch). | `services/llm/__init__.py → LLMProvider, get_llm` |
| PLB | **Transport** | The only per-provider code — OpenAI-compatible HTTP to Ollama/farm. Any failure returns `None`, so a network fault degrades to fallback, never raises into the pipeline. | `services/llm/transport.py → OpenAICompatTransport` |
| LLM | **Tasks** | Every domain method once: cache → transport → schema-validate → one retry → validators → ledger → deterministic fallback. Splits fast/deep/vision brains per task. | `services/llm/tasks.py → OllamaTasksProvider._run_structured` |
| PLB | **Schemas** | Pydantic output contracts (shape guardrails): enum patterns (`dir ∈ threat/watch/fav`), length caps, `cites` required so an ungrounded synthesis is invalid. | `services/llm/schemas.py` |
| DET | **Validators** | Semantic guardrails — `numbers_grounded`: every figure the model writes must appear in the evidence, or the output is rejected. The anti-hallucination check. | `services/llm/validators.py → numbers_grounded` |
| PLB | **Cache & ledger** | The `llm_runs` table — dedupes identical calls (template-versioned hash) and records every call (prompt hash, output, latency, validators) as an audit trail. | `services/llm/cache.py → input_hash, lookup, record` |
| DET | **Stub — the deterministic floor** | Rule-based implementation of all 7 verbs, KSSL-framed (not lorem). The whole engine runs offline on this; every fallback lands here. | `services/llm/stub.py → StubLLMProvider` |

> Also present: `services/llm/providers_legacy.py` (PLB) — Anthropic / OpenRouter providers, kept for back-compat.

---

## 4. Assets & enrichment (opt-in / external)

| Tag | Logic | Goal | Location |
|-----|-------|------|----------|
| LLM | **Multimodal analysis** | Vision brain: caption crawled images, recognize equipment, extract spec rows from PDFs into tender requirements. Opt-in via `/ops/analyze-assets`; no-ops if no vision model. | `services/multimodal.py → analyze_document_assets` |
| PLB | **Asset client** | Fetch blob bytes for a document's `s3://mallory-raw/…` path — straight from MinIO, or via the crawler proxy when MinIO isn't configured. | `services/asset_client.py` |
| DET | **Patent sync** | Pull real patents (SerpApi → USPTO → keyless Google) into `srv_patents`, replacing seed fiction. External API, deterministic mapping. | `services/patent_sync.py` |
| LLM | **Assistant** | Mallory chat + CEO report — grounded strictly over the serving tables (the client computes nothing; L2 answers from `srv_*`). | `services/assistant.py → answer` |

---

## 5. Data model — 4 namespaces + the ledger

The one-way contract: crawler writes `stg_*`, the pipeline computes `srv_*`, and L3 reads **only** `srv_*`.

| Namespace | Goal | Location |
|-----------|------|----------|
| `ref_*` | The static "vs KSSL" baseline — competitors, products, specs, matchups — loaded from seed JSON. | `models/reference.py` |
| `stg_*` | The landing zone — what the Ingest API writes from crawler records, before processing. | `models/staging.py` |
| `srv_*` | The **only** tables the L3 client reads — pre-computed cards, details, metrics, evidence. | `models/serving.py` |
| `kg_*` | Nodes/edges + the prebuilt alliance payload and graph insights the Network view renders. | `models/graph.py` |
| `llm_runs` | Cache, idempotency guard, and explainable-AI audit trail for every model call. | `models/llm_ops.py` |

---

## 6. Doors & boot — API surface + plumbing

| Tag | Logic | Goal | Location |
|-----|-------|------|----------|
| PLB | **Ingest API (A)** | Interface A — the L1→L2 door. Validates crawler bundles and writes `stg_*`. | `api/ingest.py` · `contracts/ingest.py` |
| PLB | **Serving API (B)** | Interface B — the read-only L2→L3 door. Serves cards, details, `/explain`, nav counts. | `api/serving.py` · `contracts/serving.py` |
| PLB | **Ops · Graph · Assistant · Dashboard** | Trigger the pipeline & inspect status (ops); read-only graph lookups (graph); write-back Q&A (assistant); live HTML dashboard + JSON feed. | `api/ops.py` · `api/graph.py` · `api/assistant.py` · `api/dashboard.py` |
| PLB | **App factory · Config · DB** | Assemble the FastAPI app + scheduler loop; typed settings with the one-line `LLM_TARGET` switch; SQLAlchemy engine, session & `get_db` dependency. | `main.py` · `config.py` · `db.py` |

---

*Mallory L2 · deterministic-first, LLM-assisted · numbers & rankings never model-computed · every card traceable via `/explain`.*
