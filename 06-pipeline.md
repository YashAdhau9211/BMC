# 06 — Pipeline: `runner.process_pending`

**Purpose:** the single callable that sequences the whole engine — turn pending `stg_*` rows into
`srv_*` rows, then recompute ranks, metrics, and the graph. Idempotent (only `proc_status='received'`
rows are processed; publishing upserts), so a re-run is safe.
**Called from:** `POST /ops/process`, the in-process scheduler (`main._scheduler_loop`), and the
`run_pipeline`/`demo_seed` scripts.

`pipeline/__init__.py` is not shown separately — the package holds only `runner.py`.

## `pipeline/runner.py` — line by line

**L1-6** Docstring: idempotent; in production these stages are event-driven workers, but here one
callable keeps the flow simple and testable.
**L10** `from dataclasses import dataclass` — for the result type. **L12-13** `select`, `Session`.
**L15** import the five staging models this runner queries directly. **L16-25** import the services
it orchestrates (`corroboration`, `domain_pipeline`, `extraction`, `graph_analytics`,
`graph_builder`, `metrics`, `signal_pipeline`, `tender_scoring`). **L26** `LLMProvider`, `get_llm`.

**L29-35 `PipelineResult`** — a dataclass of the five per-domain processed counts (signals,
tenders, partnerships, geo, innovation).

**L38-94 `process_pending(db, llm=None)`** — the orchestration, in strict order:
- **L41** `llm = llm or get_llm(db=db)` — **db-bound** so the cache + `llm_runs` ledger record every
  call (a prior bug: the scheduler passed a db-less llm → no ledger; this fixes it).
- **L45-46** `extraction.extract_pending(db, llm)` then `db.flush()` — **ST-1**: bare documents
  (the crawler's normal mode) → typed staging records. LLM-primary with regex fallback; stub/offline
  ⇒ pure regex. Runs first so the rows the rest of the pipeline consumes exist.
- **L50** `corr = corroboration.corroboration_counts(db)` — independent-source counts across **all**
  signals (not just this batch), so re-runs recompute as new sources arrive.
- **L52-56 QUERY + loop:** `select(StgSignal).where(proc_status=='received')` → for each,
  `signal_pipeline.process_signal(db, llm, ss, corroboration=corr.get(ss.id, 1))`. Writes
  `srv_signals` + `srv_signal_details` + evidence.
- **L58-62 QUERY + loop:** pending `StgTender` → `tender_scoring.process_tender(db, llm, st)`.
  Writes `srv_tenders` + `srv_tender_matches` (the flagship traced in `05-services.md`).
- **L64-68** pending `StgPartnership` → `domain_pipeline.process_partnership`.
- **L70-72** pending `StgGeo` → `domain_pipeline.process_geo`.
- **L74-78** pending `StgInnovation` → `domain_pipeline.process_innovation`.
- **L82** `db.flush()` — push the merges so the rank/metrics/graph queries below see the freshly
  published rows (the session runs with `autoflush=False`, so this is required).
- **L83** `signal_pipeline.recompute_ranks(db)` — S-10: rank all cards per pillar.
- **L84** `metrics.build_overview_metrics(db)` — S-11: the per-pillar metric strip.
- **L85-87** `graph_builder.rebuild_graph(db)` — full deterministic projection of `ref_*`/`srv_*` →
  `kg_*` (a `ponytail:` note flags incremental appends as a future optimization).
- **L88** `graph_analytics.run_analytics(db)` — the hidden-pattern miners + alliance payload.
- **L90** `db.commit()` — one transaction for the whole run.
- **L91-94** return the `PipelineResult` with the counts of what was processed.

### Ordering matters
Extraction must run first (it creates the typed rows). Corroboration is computed before signals
(its count feeds each signal's confidence). Ranks/metrics/graph run **after** all publishes and a
flush, because they read the serving rows the per-domain stages just wrote. The single final
`commit` makes the whole batch atomic.

### Where each `srv_*` table gets written (quick map)
| Serving table | Written by (in this run) |
|---------------|--------------------------|
| `srv_signals`, `srv_signal_details` | `signal_pipeline.process_signal` |
| `srv_tenders`, `srv_tender_matches` | `tender_scoring.process_tender` |
| `srv_partnerships` | `domain_pipeline.process_partnership` |
| `srv_geo_entries` | `domain_pipeline.process_geo` |
| `srv_innovation` | `domain_pipeline.process_innovation` |
| `srv_evidence` | every publish path (via `evidence.write_evidence`) |
| `srv_overview_metrics` | `metrics.build_overview_metrics` |
| `kg_nodes`, `kg_edges` | `graph_builder.rebuild_graph` |
| `srv_graph_insights`, `srv_alliance_graph` | `graph_analytics.run_analytics` |

Not in `process_pending` (run via their own `/ops` endpoints or scripts): `srv_matchups`
(`matchup_synthesis`), `srv_competitor_synthesis` (`competitor_synthesis`), `srv_field_patterns`
(`field_patterns`), `srv_patents` (`patent_sync`), `stg_asset_analysis` (`multimodal`).
