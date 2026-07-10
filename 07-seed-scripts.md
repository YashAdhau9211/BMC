# 07 — Seed loader + CLI scripts

The static baseline loader (`seed/`) and the operational entrypoints (`scripts/`). These aren't in
the request path — they populate `ref_*`, create tables, and drive the pipeline for dev/demo.

---

## `seed/loader.py` — bundled JSON → `ref_*` (idempotent, upsert by PK)

**Purpose:** load `seed_data/*.json` into the reference tables (and the *fallback* seed rows for a
few serving tables). Idempotent via `db.merge` (upsert by primary key).
**Reads:** the seed JSON files. **Writes:** `ref_categories`, `ref_competitors`,
`ref_kssl_products`, `ref_competitor_products`, `ref_tech_domains`, `ref_countries`,
`ref_product_specs`, `ref_matchups`, plus seed-fallback rows for `srv_competitor_synthesis`,
`srv_field_patterns`, `srv_patents`, and (via the S-22 engine) `srv_matchups`.

**L29-38** `_CATEGORY_NAMES` — slug → display name for the eight categories.
**L41-42 `_slug`** / **L45-49 `_read`** — id-safe slug; read a seed JSON file (`{}` if missing).
**L52-178 `load_all(db, seed_dir=None)`** — the whole seed, section by section:
- **L56-60 categories** — `watchlist_products.json` `categories` → `db.merge(RefCategory)`.
- **L62-78 competitors** — the `anchor` (KSSL: `is_anchor=True`, `priority="P0"`, dir `fav`) + each
  rival from `watchlist_entities.json` → `db.merge(RefCompetitor)`.
- **L80-85 KSSL products** — `kssl_products` → `db.merge(RefKsslProduct)`.
- **L87-97 competitor products** — `competitor_products` keyed by competitor id; synthesize a
  product id `{comp}_{slug(name)}` → `db.merge(RefCompetitorProduct)`.
- **L99-103 tech domains** — `watchlist_tech_domains.json` → `db.merge(RefTechDomain)` (their
  `keywords` are what `extraction`/`patent_sync` classify against).
- **L105-109 countries** — `watchlist_tenders.json` `target_countries` → `db.merge(RefCountry)`.
- **L111-121 KSSL specs** — `kssl_product_specs.json` → `db.merge(RefProductSpec)` with
  `product_side="kssl"` (the rows tender scoring compares against).
- **L123-137 matchups** — `matchups.json` → `db.merge(RefMatchup)`, then **L136-137**
  `matchup_synthesis.recompute_all(db)` to build the `srv_matchups` serving rows with template
  verdicts (LLM verdicts come later via `/ops/recompute-matchups`).
- **L139-153 competitor synthesis (seed = estimate fallback)** — `competitor_synthesis.json`; for
  each, **skip if the S-23 engine already published a `sourced` row** (never overwrite real intel);
  else `db.merge(SrvCompetitorSynthesis(..., provenance="estimate"))`.
- **L155-164 field patterns (seed fallback)** — only if no `sourced` pattern exists: replace the
  `srv_field_patterns` with the seed set.
- **L166-175 patents (sample)** — `patents.json` → `db.merge(SrvPatent(..., provenance="estimate"))`
  until `patent_sync` replaces them with real filings.
- **L177** `db.commit()`. **Output:** a `{section: count}` dict.

**Provenance discipline:** everything the loader writes to a serving table is `estimate` and is a
*fallback only* — the real engines (`matchup_synthesis`, `competitor_synthesis`, `field_patterns`,
`patent_sync`) overwrite these with `sourced` rows and are never clobbered by a re-seed.

`seed/__init__.py` (**L1**) is docstring only.

---

## `scripts/` — operational entrypoints

Each is a `python -m mallory_engine.scripts.<name>` CLI. `scripts/__init__.py` (**L1**) is docstring
only.

### `init_db.py`
**L5-6** import `models` (registers every table on `Base.metadata`) + `Base`/`engine`. **L9-11
`main()`** — `Base.metadata.create_all(engine)`; print the table count. Dev convenience (production
uses Alembic).

### `load_seed.py`
**L9-12 `main()`** — open a session, `load_all(db)`, print the seeded counts. Wraps
`seed.loader.load_all`.

### `run_pipeline.py`
**L9-12 `main()`** — open a session, `process_pending(db)`, print processed signal/tender counts.
The CLI form of `POST /ops/process`.

### `demo_seed.py` — one-command live demo DB
**Purpose:** drop+recreate all tables, seed `ref_*`, insert a hand-built set of **real crawled
events** (documents/signals/partnerships/geo/tenders mirroring the crawler's actual catches), run
the full pipeline, then synthesis + field patterns. The fastest way to see the whole engine end to
end on SQLite with no Docker.
**L33-53 `DOCS`** / **L55-71 `SIGNALS`** — the sample corpus (K9 Vajra orders, Adani/UAV, KNDS
CAESAR, NIBE licensing, Solar loitering munitions, two 155mm RFPs).
**L74-137 `main()`** — reconfigure stdout to UTF-8 (₹/é in defence data); `drop_all`+`create_all`;
`loader.load_all`; add the sample `Stg*` rows (note tender #1 carries the `requirement_fields`
`System/Range/Weight` that tender scoring parses into slots); commit; `runner.process_pending`;
retry `synthesize_competitor("LT")` up to 4× (generation is sampled, fail-safe keeps rows);
`refresh_field_patterns`; commit.

### `mock_feeder.py` — pretend to be the crawler
**Purpose:** POST sample page envelopes to the running Ingest API, then trigger `/ops/process`. Lets
L2/L3 be built before the real crawler exists (the samples are in exact crawler-output shape).
**L18-19** `API` base + default sample file. **L22-34 `main()`** — read the sample envelopes; for
each, `POST /ingest/v1/page`; then `POST /ops/process` and print the result.

### `sync_patents.py` — run the real patent connector
**L12-18 `_selfcheck()`** — offline pure-function checks on `_jurisdiction`/`_status`. **L21-31**
`--check` runs the selfcheck and exits; otherwise open a session, `patent_sync.sync_patents(db)`,
then `graph_builder.rebuild_graph(db)` (so the new `competitor -[filed]-> patent` edges show),
commit.

### `try_llm.py` — LLM smoke driver
**Purpose:** run one real crawler record through the configured provider and show it working — plus,
with a DB reachable, demonstrate the `llm_runs` ledger and a **cache hit** on a repeated call.
**L23** `NDJSON` — path to the crawler's `ingested.ndjson`. **L26-36 `_first_event`** — the first
crawler title (or a built-in sample). **L39-49 `_try_db`** — open a session if a DB is reachable,
else run ledger-less. **L51-87 `main()`** — resolve the llm; `classify_signal` + `enrich_signal`
on the event and print them; if a DB is present, print the ledger row count, repeat the classify,
and confirm the second call was a cache hit (no new ledger row).

---

## That's the whole engine

Across `00`–`07` every source file in `layer2-data-engine/src/mallory_engine` is covered: the
tables and their schema, the queries each stage runs, the raw input → output of every function, the
referenced functions, and the logic. Start at `00-overview.md` for the map; follow the data from
`api/ingest.py` (writes `stg_*`) through `pipeline/runner.py` (the compute) to `api/serving.py`
(reads `srv_*`).
