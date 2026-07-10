# L2 Data Engine — Code Walkthrough (every-line)

This folder documents **every source file** in `layer2-data-engine/src/mallory_engine`,
line by line: what each line does, which **table/schema** it reads or writes, the exact
**query**, the **raw input → output** of each function, the **referenced functions** it
calls, and the **logic**.

Read the docs in this order — it follows the data flow:

| Doc | Layer | Files covered |
|-----|-------|---------------|
| `00-overview.md` | (this file) architecture + data flow | — |
| `01-foundation.md` | DB engine + config | `db.py`, `config.py` |
| `02-models.md` | **The tables + their schema** | `models/*.py` |
| `03-contracts.md` | API request/response schemas | `contracts/*.py` |
| `04-api.md` | HTTP endpoints (writes staging, reads serving) | `api/*.py`, `main.py` |
| `05-services.md` | The processing brain | `services/*.py` |
| `06-pipeline.md` | Orchestration | `pipeline/runner.py` |
| `07-seed-scripts.md` | Seed loader + CLI scripts | `seed/`, `scripts/` |

---

## What L2 actually is

L2 (Layer 2, "the data engine") sits between the **crawler** (Layer 1, which scrapes the
web) and the **client UI** (Layer 3). Its one job:

> Take raw scraped documents, turn them into **typed intelligence records**, score/enrich
> them, and write **pre-computed, denormalized rows** that the UI can render with zero math.

It is a FastAPI app + a processing pipeline over a SQL database (Postgres in prod, SQLite
for dev). The LLM (Ollama / Anthropic / OpenRouter, or a deterministic "stub") is used only
where a rule can't do the job — and every LLM call is logged and cached.

---

## The four table families (namespaces)

Every table's name prefix tells you its role. This is the single most important thing to
understand — the whole engine is "move data from one prefix to the next."

| Prefix | Namespace | Who writes it | Who reads it | Meaning |
|--------|-----------|---------------|--------------|---------|
| `ref_*` | **reference** | admin / seed JSON | services | Static "vs KSSL" baseline: categories, competitors, KSSL products, their specs. Loaded once from `seed_data/`. |
| `stg_*` | **staging** | the Ingest API (from crawler) + L2 extraction | services (never the client) | Raw + typed records mid-processing. Append-only. Each row walks a state machine. |
| `srv_*` | **serving** | the pipeline services | **only** the Layer 3 client | Denormalized, pre-computed. Every value the UI shows is a literal column here. |
| `kg_*` | **knowledge graph** | the graph builder | graph endpoints + serving projections | Nodes/edges projected from `ref_*`/`srv_*`. Fully rebuilt each run. |
| `llm_runs` | **ledger** | the LLM task layer | the LLM cache | Audit + cache of every structured LLM call. |

### KSSL / "the anchor"
`KSSL` is the home company (the "anchor" competitor, `is_anchor=True`). Everything is scored
*from KSSL's point of view* — "is this tender a good fit for **us**?", "does our product beat
theirs?". You'll see `ANCHOR` all over the LLM prompts.

---

## The processing state machine

Every staging row carries `proc_status`, starting at `received`. The docstring on
`models/staging.py` names the full walk:

```
received → resolved → classified → enriched → published
```

A row is only picked up by the pipeline while `proc_status='received'`; once its serving row
is written, it flips to `published`. This makes the whole pipeline **idempotent** — re-running
it does nothing to already-published rows (see `pipeline/runner.py`).

---

## End-to-end data flow (one document's journey)

```
        Layer 1 crawler
              │  POST /ingest/documents  (raw page: title, text, images, tables…)
              ▼
     ┌──────────────────┐
     │  api/ingest.py   │  writes ─────────────►  stg_documents  (raw doc, proc via extraction)
     └──────────────────┘                         (+ maybe stg_signals/tenders/… if L1 pre-typed)
              │
              │  pipeline/runner.py :: process_pending()   (scheduler or POST /ops/run)
              ▼
     ┌────────────────────────────────────────────────────────────────────┐
     │ services/extraction.py   bare stg_documents → typed stg_* records   │
     │ services/*_pipeline.py    each stg_* row  → srv_* row(s)            │
     │   e.g. tender_scoring.py  stg_tenders  → srv_tenders + srv_tender_matches
     │        signal_pipeline.py stg_signals  → srv_signals + srv_signal_details
     │ services/graph_builder    ref_*/srv_*  → kg_nodes/kg_edges          │
     │ services/metrics          srv_*        → srv_overview_metrics        │
     └────────────────────────────────────────────────────────────────────┘
              │
              │  GET /serving/*   (api/serving.py — read-only)
              ▼
        Layer 3 client UI  (renders literal columns; no computation)
```

Reference lookups (`ref_kssl_products`, `ref_product_specs`, …) are read *during* processing
to supply the "vs KSSL" baseline — they are inputs, never written by the pipeline.

---

## How to read the per-line docs

Each file gets:
- a one-line **purpose**,
- **inputs / outputs / tables touched** at a glance,
- then **every line** annotated as: **`L<n>`** `code` — explanation.

Trivial lines (`from __future__ import annotations`, blank lines) are annotated once per file
and then skipped by reference, to keep the signal high — but no logic line is skipped.
