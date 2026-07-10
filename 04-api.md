# 04 — API: the HTTP endpoints

Two hard interfaces plus internal controls:
- **`api/ingest.py`** — Interface A (L1→L2): writes `stg_*` (`proc_status='received'`).
- **`api/serving.py`** — Interface B (L2→L3): read-only over `srv_*`; filters → `WHERE`/`ORDER BY`.
- **`api/dashboard.py`** — one consolidated read for the live HTML dashboard.
- **`api/graph.py`** — read-only graph lookups (ego/path BFS over `kg_edges`).
- **`api/ops.py`** — run the pipeline / synthesis engines; processing-state counts.
- **`api/assistant.py`** — chat + CEO report write-back proxies.
- **`main.py`** — the FastAPI app factory + optional in-process scheduler.

Every handler takes `db: Session = Depends(get_db)` — that's the per-request session from
`db.py`. `select(...)` builds a SQLAlchemy query; `db.scalars(stmt).all()` returns ORM rows;
`db.scalar(stmt)` returns one value; `db.get(Model, pk)` is a primary-key lookup.

---

## `api/__init__.py`
**L1-6** Docstring only: names the three interfaces (ingest / serving / ops).

---

## `api/ingest.py` — write staging (Interface A)

**Purpose:** validate the crawler's payload, upsert the document, insert typed records (with
dedup), all at `proc_status='received'`. **Processing is a separate step** — ingestion just lands
data. **Tables written:** `stg_documents` + the six `stg_*` record tables.
**L9-17** Imports: `hashlib` (doc-id hash), FastAPI router bits, `ValidationError`, `select`,
`Session`. **L19-28** Import all the ingest contracts. **L29-30** `get_db`, `models.staging as stg`.
**L32** `router = APIRouter(prefix="/ingest/v1", tags=["ingest"])` — every route below is under
`/ingest/v1`.

**L35-36** `_doc_id(url)` → `"doc_" + sha1(url)[:12]`. **Deterministic id from the URL** — the
same URL always yields the same `stg_documents.id`, which is what makes upsert idempotent.
**L39-40** `_now()` → timezone-aware UTC now (used for `received_at`).

**L43-63 `upsert_document(db, d: DocumentIn) -> str`** — the core writer.
- **L44** `did = _doc_id(d.url)`.
- **L45-55** `fields = dict(...)` — every column value, with the nested pydantic objects
  serialized via `.model_dump(mode="json")` (so `images`/`attachments`/`screenshot`/`tables`/
  `entities_detected` land as plain JSON in the JSONB columns).
- **L56** `existing = db.get(stg.StgDocument, did)` — PK lookup.
- **L57-59** If it exists, **update** every field via `setattr` (re-crawl refreshes the row).
- **L60-61** Else `db.add(StgDocument(id=did, received_at=_now(), dedup_status="new", **fields))`.
- **L62** `db.flush()` — force the INSERT now so child records can FK-reference this document.
- **L63** returns `did`. **Input:** a `DocumentIn`. **Output:** the doc id + a row in
  `stg_documents`.

**L66-80 `_ingest_signal(db, did, r)` → bool** — insert one signal, deduped.
- **L67-71 QUERY:** `select(StgSignal).where(document_id==did AND event_summary==r.event_summary)`
  — same doc + same summary ⇒ duplicate.
- **L72-73** if dup → return `False` (nothing inserted).
- **L74-79** else `db.add(StgSignal(... all crawler fields ...))`; return `True`. The bool feeds
  the per-type count in `ingest_page`.

**L83-100 `_ingest_tender(db, did, r)` → bool** — insert one tender, deduped.
- **L84** `key = r.source_ref or r.title` — prefer the stable source ref.
- **L85-90 QUERY:** `select(StgTender).where(document_id==did AND (source_ref==key OR
  title==r.title))` — dup if either the ref or the title already exists for this doc.
- **L93-99** else insert, serializing `requirement_fields` to JSON. Returns `True`. **This is the
  row `tender_scoring.process_tender` later consumes.**

**L103-107 `_exists(db, model, **filters)` → bool** — generic "is there a row matching these
column==value filters?" Builds `select(model)` + a `.where` per filter; returns
`db.scalar(stmt) is not None`. Used by the remaining record types' dedup.

**L110-131** `_ingest_partnership` / `_ingest_geo` / `_ingest_innovation` /
`_ingest_company_event` — each: dedup via `_exists` on its natural key
(partner_name / product_name+country / title / headline), else
`db.add(Model(document_id=did, **r.model_dump(exclude={"document_id"})))`. (`exclude` avoids
passing `document_id` twice.) These return `None` (no count needed).

### Routes
**L134-150 `POST /ingest/v1/page`** (`ingest_page`) — the atomic one-shot.
- **Input:** a `PageEnvelopeIn` (validated → 422 on malformed). **L136** upsert the document.
- **L137-140** count inserted signals/tenders via `sum(_ingest_*(...))` (each returns 0/1).
- **L141-148** insert partnerships/geo/innovation/company_events.
- **L149** `db.commit()`. **Output:** `{"document_id": did, "ingested": counts}`.

**L153-157 `POST /ingest/v1/document`** (`ingest_document`) — upsert a document only; commit;
return its id.

**L162-169** `_DISPATCH` — maps a `record_type` string → (`contract model`, `ingest fn`). Powers
the per-record route.

**L172-188 `POST /ingest/v1/{record_type}`** (`ingest_bundle`) — the crawler's forward shape
`{document, record}`.
- **L174-175** unknown type → 422 `unknown_record_type`.
- **L176-178** empty `main_text` → 422 `rule1_empty_main_text` (the crawler contract's rule 1).
- **L179-184** validate the document + record; a `ValidationError` → 422 `rule3_invalid_record`
  with the pydantic error detail.
- **L185-188** upsert doc, call the mapped ingest fn, commit, return `{"accepted": True,
  "document_id": did}`.

---

## `api/serving.py` — read serving (Interface B)

**Purpose:** read-only. Filters map to `WHERE`/`ORDER BY` on **pre-computed** columns; the client
computes nothing. **Tables read:** the `srv_*` tables + `ref_competitors`. **Writes:** none.
**L9-53** Imports: `httpx` (asset proxy), FastAPI bits, `func`/`select`, all the serving DTOs,
`RefCompetitor`, `fetch_asset`, and every `Srv*` model. **L55** `router = APIRouter(prefix=
"/api/v1", tags=["serving"])`.

**L58-80 `GET /api/v1/signals`** (`list_signals`) — paginated feed.
- **Inputs (query params):** `pillar` (default `competitive`), `filter` (default `all`),
  `company`, `page≥1`, `size 1-100`.
- **L67-68** base + count statements filtered by `pillar`. **L69-74** add `dir==filter` and
  `company==company` filters when set (to **both** the data and count query).
- **L76** `total = db.scalar(count_stmt)`.
- **L77-79 QUERY:** `stmt.order_by(SrvSignal.rank).offset((page-1)*size).limit(size)` — **the
  rows come out pre-sorted by the precomputed `rank`.**
- **L80** wrap rows as `SignalCard`s in a `Page`. **Output:** `Page[SignalCard]`.

**L83-88 `GET /api/v1/signals/{signal_id}/detail`** — `db.get(SrvSignalDetail, signal_id)`; 404
if missing; return `SignalDetail`.

**L91-92** `_CONF_MODEL = {"signal": SrvSignal}` — which srv table carries confidence/provenance
per explainable kind.

**L95-133 `GET /api/v1/explain/{target_kind}/{target_id}`** (`explain`) — **the "why this?"
endpoint.**
- **L102-106 QUERY:** `select(SrvEvidence).where(target_kind==.. AND target_id==str(id))
  .order_by(id)` — every evidence link for that row.
- **L108-117** group links **by field** into `FieldExplanation` objects, each accumulating
  `EvidenceRef`s. (First link for a field sets its `method`.)
- **L119-122** build the `ExplainResponse` with the evidence count + grouped fields.
- **L123-130** if the kind has a confidence-owning table, `db.get` that row and copy
  `provenance`/`confidence`/`confidence_band`/`confidence_parts` onto the response.
- **L131-132** 404 only if there's neither evidence nor a conf model. **Output:**
  `ExplainResponse`. Works identically for rule- and LLM-produced rows (that's the point).

**L136-141 `GET /api/v1/overview/{pillar}/metrics`** — `db.get(SrvOverviewMetrics, pillar)`; 404
if not computed; return `OverviewMetrics`.

**L144-152 `_tender_with_matches(db, t)`** — helper that turns one `SrvTender` into a full
`TenderCard`.
- **L145-149 QUERY:** `select(SrvTenderMatch).where(tender_id==t.id).order_by(fit_pct.desc())` —
  its scored product matches, best first.
- **L150-151** validate the tender into a `TenderCard`, then attach `card.matches = [...]`.
  **This is where the nested `matches` list on the DTO gets filled.**

**L155-173 `GET /api/v1/tenders`** (`list_tenders`) — the tender list.
- **Inputs:** `filter` (`all|go|maybe|pass|closing`), `category`, `sort` (`deadline|value`).
- **L162-168** `filter in (go,maybe,pass)` → `WHERE lean==filter`; `closing` → `WHERE
  status=='closing'`; optional `category` → `WHERE category==..`.
- **L169-172** order by `value_usd desc nullslast` (sort=value) else `deadline_date asc
  nullslast`.
- **L173** returns a `TenderCard` per row, **each expanded via `_tender_with_matches`** (so the
  list already carries every product match). **Output:** `list[TenderCard]`.

**L176-181 `GET /api/v1/tenders/{tender_id}`** — `db.get(SrvTender, id)`; 404; expand via
`_tender_with_matches`.

**L184-204 `GET /api/v1/nav/counts`** — left-rail counts. Inner `n(model, *where)` runs
`select(func.count()).select_from(model)` + optional wheres. Returns a dict of counts for
competitive/market/technology signals, matchups, partnerships, geo, tenders, innovation, patents.

**L207-214 `GET /api/v1/competitors`** — `select(RefCompetitor).order_by(name)`; returns
`[{id,name,hq,dir,is_anchor}]`. (Reads the **reference** table directly — the client's competitor
picker.)

**L220-224 `_matchup_with_specs(db, m)`** — like the tender helper: query
`select(SrvMatchupSpec).where(matchup_id==m.id)`, attach as `card.specs`.

**L227-240** `GET /api/v1/matchups` (optional `category` filter, order by `edge_score`, each
expanded with specs) and `GET /api/v1/matchups/{id}` (get + expand). **Output:** `MatchupCard`(s).

**L246-255 `GET /api/v1/geo`** — `select(SrvGeoEntry)` + optional `competitor`/`country` filters,
order by `country`. **Output:** `list[GeoEntry]`.

**L261-269 `GET /api/v1/partnerships`** — `select(SrvPartnership)` + optional `competitor`, order
by `competitor_name`. **Output:** `list[PartnershipCard]`.

**L275-280 `GET /api/v1/innovation`** — `select(SrvInnovation)` + optional `domain` (→
`tech_domain_id`). **Output:** `list[InnovationCard]`.

**L286-295 `GET /api/v1/patents`** — `select(SrvPatent)` + optional `competitor`/`domain`.
**Output:** `list[PatentCard]`.

**L301-306 `GET /api/v1/competitors/{id}/synthesis`** — `db.get(SrvCompetitorSynthesis, id)`; 404;
return `CompetitorSynthesisDTO`.

**L309-312 `GET /api/v1/field-patterns`** — `select(SrvFieldPattern).order_by(ord)`. **Output:**
`list[FieldPatternDTO]`.

**L315-335 `GET /api/v1/asset-proxy`** — relays a binary asset (image/PDF/screenshot) from the
crawler/MinIO by its `s3://...` `storage_path`, via `fetch_asset` (see `05-services.md`). Maps
upstream 4xx → same status, connection error → 502. Returns raw bytes as
`application/octet-stream`. (This is the one serving route that isn't a `srv_*` read.)

---

## `api/dashboard.py` — one consolidated live read

**Purpose:** a single endpoint returning *everything* the live HTML dashboard renders, from the
running DB (so it reflects current state). **Reads:** most `srv_*` + `kg_*` tables. **Writes:**
none.
**L34-51 `_find_static()`** — locate the `static/` dir across source-tree, container CWD, and an
`MALLORY_STATIC_DIR` env override; first candidate containing `dashboard_live.html` wins.
`_STATIC` caches it.
**L54-70 `_layout(nodes, edges)`** — a **deterministic** circular-by-kind graph layout computed
in pure Python (no numpy/networkx at request time): group node ids by `kind`, place each kind on
its own ring radius, spread nodes around the ring by index (angle offset seeded from `hash(kind)`
for reproducibility). Returns `{node_id: (x, y)}`.

**L73-154 `GET /api/v1/dashboard/data`** (`dashboard_data`) — the big read.
- **L75-79** build an `evidence` map: `select(SrvEvidence)` → group by `"{target_kind}:{target_id}"`.
- **L81-83** load all `KgNode`/`KgEdge`, compute `pos` via `_layout`.
- **L85-89** `_syn(r)` — flatten one `SrvCompetitorSynthesis` to a dict.
- **L91-153** assemble one dict with: `signals` (all, ordered by pillar+rank), `tenders` (each
  with a nested `matches` sub-query ordered by `fit_pct desc`), `matchups` (each with a nested
  `specs` sub-query), `synthesis` (non-empty theses), `field_patterns`, `insights`
  (`SrvGraphInsight` by rank), `graph` (nodes with x/y/community/degree/dir/anchor + edges),
  `evidence`, and `stats` (node/edge/signal/evidence counts). **Output:** one JSON blob → no
  N round-trips.

**L157-163 `GET /dashboard`** — serves the `dashboard_live.html` file (or a 404 text if missing).

---

## `api/graph.py` — read-only graph lookups

**Purpose:** index-lookups over `kg_*`; anything scored/ranked is precomputed. Ego/path are plain
**BFS over indexed edges, depth-capped**. **Reads:** `kg_nodes`, `kg_edges`,
`srv_alliance_graph`, `srv_graph_insights`.
**L18** `router = APIRouter(prefix="/api/v1/graph")`. **L20** `_MAX_NODES = 150` — the BFS cap.

**L23-29 `GET /alliances`** — `db.get(SrvAllianceGraph, "latest")` (the single prebuilt payload);
404 with a hint to rebuild; return `{generated_at, nodes, edges, stats}`.

**L32-42 `GET /insights`** — `select(SrvGraphInsight).order_by(rank)` + optional `kind` filter;
returns the insight cards.

**L45-51 `_neighbors(db, node_ids, rels)`** — **QUERY:** `select(KgEdge).where(src_id IN node_ids
OR dst_id IN node_ids)` + optional `rel IN rels`. The edge-expansion primitive for both BFS
endpoints.
**L54-57 `_node_dicts(db, ids)`** — `select(KgNode).where(id IN ids)` → node dicts.

**L60-93 `GET /ego`** — ego network around one node (undirected BFS, depth ≤ 3).
- **Inputs:** `node` (required), `depth` (1-3, default 2), `rels` (comma-separated filter).
- **L67-68** 404 if the node is unknown.
- **L71-83** BFS: for up to `depth` rounds, expand the `frontier` via `_neighbors`, collect
  edges, compute the next frontier (`new nodes − seen`), stop at `_MAX_NODES`.
- **L85-92** dedup edges whose both endpoints survived the cap; return `{center, nodes, edges}`.

**L96-134 `GET /path`** — shortest evidence-bearing path between two nodes.
- **L99-100** 404 if src or dst unknown.
- **L102-115** BFS with **parent tracking** (a `deque`), undirected, capped at `max_depth` (1-6).
- **L117-118** if dst unreachable → `{found: False, ...}`.
- **L120-134** walk the `parent` chain back from dst to reconstruct the hop edges, reverse them,
  return `{found: True, nodes, edges}`.

---

## `api/ops.py` — internal pipeline controls

**Purpose:** trigger the pipeline / synthesis engines and report processing-state counts (the
monitor's data). **Not client-facing.**
**L23** `router = APIRouter(prefix="/ops")`.

**L26-35 `POST /ops/process`** — `process_pending(db)` (the whole pipeline, see `06-pipeline.md`);
returns the per-type processed counts.
**L38-43 `POST /ops/rebuild-graph`** — `graph_builder.rebuild_graph` + `graph_analytics.run_analytics`,
commit, return merged counts.
**L46-50 `POST /ops/recompute-matchups`** — S-22: `matchup_synthesis.recompute_all(db,
get_llm(db=db))`, commit.
**L53-61 `POST /ops/synthesize`** — S-23 competitor synthesis, all or one via `?competitor=`.
**L64-68 `POST /ops/field-patterns`** — S-24: `field_patterns.refresh_field_patterns`.
**L71-74 `POST /ops/analyze-assets`** — multimodal captioning/spec-extraction (opt-in because the
vision model must swap into VRAM).
**L77-91 `GET /ops/status`** — inner `by_status(model)` runs `select(model.proc_status,
func.count()).group_by(proc_status)` for `StgSignal`/`StgTender`, plus serving counts. Feeds the
monitor view.

---

## `api/assistant.py` — chat + report write-backs

**L16-18 `POST /api/v1/mallory/chat`** — validates a `MalloryRequest`, delegates to
`assistant.answer(db, get_llm(), req)`, returns `MalloryResponse`.
**L21-23 `POST /api/v1/reports/ceo`** — `assistant.ceo_report(db, get_llm(), req.focus)` →
`ReportResponse`. (Compute stays in L2; the client still computes nothing.)

---

## `main.py` — the app factory + scheduler

**L19-36 `_scheduler_loop(interval_s)`** — optional micro-batch loop: every `interval_s`, open a
session and run `process_pending` **in a worker thread** (`asyncio.to_thread`, because the ORM is
sync), logging on any processed rows; swallow+log exceptions so one bad tick doesn't kill the loop.
`ponytail:` note — single in-process loop; upgrade to APScheduler + advisory lock for multiple
cadences/instances.
**L39-50 `_lifespan(app)`** — on startup, if `scheduler_enabled`, spawn the loop task; on
shutdown, cancel it cleanly.
**L53-91 `create_app()`** — build the `FastAPI` app; add CORS (origins from
`settings.cors_origin_list`); **include all six routers** (ingest, serving, assistant, ops, graph,
dashboard); add a tiny `/` index page and a `/health` JSON.
**L94** `app = create_app()` — the module-level ASGI app uvicorn serves.

---

**Next:** `05-services.md` — the processing brain (extraction, the per-domain pipelines incl.
tender scoring, the LLM layer, graph, synthesis, confidence/corroboration/evidence).
