# 02 — Models: the tables and their schema

Every table L2 uses is defined here as a SQLAlchemy ORM class (`class X(Base)` →
`__tablename__`). This is the **schema** — column name, type, nullability, and (in comments)
the allowed values. Files:

- `models/__init__.py` — registers all model modules.
- `models/reference.py` — `ref_*` static baseline.
- `models/staging.py` — `stg_*` crawler/typed records (the pipeline's input).
- `models/serving.py` — `srv_*` denormalized rows (the client's only source).
- `models/graph.py` — `kg_*` knowledge graph + its `srv_*` projections.
- `models/llm_ops.py` — `llm_runs` ledger.

## Column patterns you'll see repeatedly (explained once)

- **`from __future__ import annotations`** — string-eval type hints (top of every file).
- **`Mapped[str] = mapped_column(String, primary_key=True)`** — a typed column; `Mapped[T]` is
  the Python type, `mapped_column(...)` the SQL definition. `| None` in the type + `nullable=True`
  = optional column.
- **The autoincrement PK line**
  `id: Mapped[int] = mapped_column(BigInteger().with_variant(Integer, "sqlite"),
  primary_key=True, autoincrement=True)` — a surrogate key that is `BIGINT` on Postgres but
  `INTEGER` on SQLite (SQLite only autoincrements `INTEGER`). Appears on almost every `stg_*`/
  `srv_*` table; annotated once, referenced as **"[std autoincrement PK]"** after.
- **`JSONB`** — a JSON column (list/dict); renders as `JSON` on SQLite via the `db.py` shim.
- **`index=True`** — a DB index (the column is filtered/joined on often).
- Comments like `# threat|watch|fav` enumerate the **allowed string values** for that column.

---

## `models/__init__.py`

**L1-8** Docstring naming the namespaces: `reference→ref_*` (static seed), `staging→stg_*`
(crawler raw), `serving→srv_*` (client-facing). Importing the package registers every table on
`Base.metadata`.
**L10** `from . import graph, llm_ops, reference, serving, staging` — importing each module runs
its `class X(Base)` definitions, which is what registers the tables (needed before
`Base.metadata.create_all`). `# noqa: F401` silences "imported but unused" (the import *is* the
side effect).
**L12** `__all__` — the public module list.

---

## `models/reference.py` — the static "vs KSSL" baseline (`ref_*`)

**Purpose:** the admin-owned catalog loaded from `seed_data/` JSON. Read during processing to
supply the KSSL comparison; never written by the pipeline.
**L1** Docstring. **L5-7** Imports: `Boolean, ForeignKey, Integer, Numeric, String`, `JSONB`,
and `Mapped`/`mapped_column`. **L9** `from ..db import Base`.

### `RefCategory` → `ref_categories` (L12-15)
A product category (e.g. artillery).
**L14** `id: Mapped[str] = mapped_column(String, primary_key=True)` — string PK, e.g.
`'artillery'`.
**L15** `name` — display name.

### `RefCountry` → `ref_countries` (L18-22)
**L20** `id` — country code PK. **L21** `name`. **L22** `region` — optional grouping.

### `RefTechDomain` → `ref_tech_domains` (L25-29)
Technology domains for the innovation stream.
**L27** `id` PK (e.g. `'artillery'`). **L28** `name`. **L29** `keywords: JSONB` — match terms
used to bucket documents into this domain.

### `RefCompetitor` → `ref_competitors` (L32-40)
One competitor company — **including KSSL itself** (the anchor).
**L34** `id` PK (e.g. `'LT'`). **L35** `name`. **L36** `aliases: JSONB` — alternate names for
entity resolution. **L37** `hq_country`. **L38** `threat_level` — `threat|watch|fav`.
**L39** `is_anchor: Boolean default=False` — **true only for KSSL**; this flag is how code
knows which row is "us". **L40** `priority` — `P1|P2|P3`.

### `RefKsslProduct` → `ref_kssl_products` (L43-48)
A product KSSL sells. **This is queried by `tender_scoring.process_tender` to enumerate what to
score a tender against.**
**L45** `id` PK (e.g. `'ATAGS'`). **L46** `name`. **L47** `category_id:
ForeignKey("ref_categories.id")` — links to `ref_categories`; the tender scorer filters KSSL
products by this against `stg_tenders.category_hint`. **L48** `aliases: JSONB`.

### `RefCompetitorProduct` → `ref_competitor_products` (L51-57)
A competitor's product (the "them" side of matchups).
**L53** `id` PK (e.g. `'CAESAR6x6'`). **L54** `competitor_id: FK→ref_competitors.id`.
**L55** `name`. **L56** `category_id: FK→ref_categories.id`. **L57** `aliases: JSONB`.

### `RefMatchup` → `ref_matchups` (L60-78)
Admin-curated pairing: which KSSL product is benchmarked vs which competitor product. The
serving `srv_matchups` are **recomputed** from these + `ref_product_specs` (never hand-written).
**L68** `id` PK (e.g. `'ATAGS__caesar_6x6'`). **L69** `kssl_product_id`. **L70** `kssl_name`.
**L71** `comp_name`. **L72** `comp_by` — the competitor company. **L73** `country`.
**L74** `category_id`. **L76** `specs: JSONB` — paired spec rows exactly as curated
(`[{label,kssl,comp,kssl_num?,comp_num?,better?,highlight?}]`). **L77** `adv_kssl: JSONB` /
**L78** `adv_comp: JSONB` — curated context/advantage lines for each side.

### `RefProductSpec` → `ref_product_specs` (L81-93)
**One spec row per (product, attribute)** — works for both KSSL and competitor products. This
is the table `tender_scoring._kssl_specs` and the matchup engine query.
**L85** `id: Integer primary_key autoincrement` — surrogate PK.
**L86** `product_id: String index=True` — which product (KSSL or competitor id).
**L87** `product_side: String` — `'kssl' | 'competitor'`; the discriminator that says which
catalog `product_id` refers to.
**L88** `spec_label` — the attribute name, e.g. `'Max range'`.
**L89** `value_text` — the raw display value, e.g. `'40+'`.
**L90** `value_num: Numeric` — the parsed numeric, e.g. `40.0` (what scoring compares).
**L91** `unit` — e.g. `'km'`. **L92** `polarity` — `higher_better|lower_better` (which
direction is "good"). **L93** `is_highlight: Boolean` — flag a headline spec.

---

## `models/staging.py` — crawler + typed records (`stg_*`)

**Purpose:** what the Ingest API writes from crawler records, plus the typed records L2's
extraction stage derives. **Append-only, never read by the client.** Each row walks
`received → resolved → classified → enriched → published` via its `proc_status`.
**L1-5** Docstring (states the state machine). **L11-13** Imports: many SQL types + `JSONB` +
`Mapped`/`mapped_column`. **L15** `from ..db import Base`.

### `StgDocument` → `stg_documents` (L18-44)
The **raw scraped page** — the root record everything else hangs off (FK target).
**L20** `id: String primary_key` — e.g. `'doc_8a91'`. **L21** `url: String unique` — canonical
URL, the dedup key. **L22** `content_hash: String index` — hash of body, second dedup key.
**L23** `source_id` / **L24** `source_tier: Integer` — provenance + trust tier of the source.
**L25** `title`. **L26** `author`. **L27** `published_at: DateTime(tz)`. **L28**
`date_precision`. **L29** `language`. **L30** `access`. **L31** `main_text: Text` — the article
body. **L32** `main_text_en: Text` — English translation if the original wasn't English.
**L33** `summary: Text`. **L34** `images: JSONB` — captured image refs. **L35**
`attachments: JSONB` — e.g. PDFs. **L36** `screenshot: JSONB`. **L37** `tables: JSONB` —
extracted tables. **L38** `entities_detected: JSONB` — crawler-detected entities.
**L39** `fetched_at` / **L40** `received_at: DateTime(tz)` (non-null) — when fetched vs when L2
received it. **L41** `dedup_status` — `new|duplicate_of:<id>`. **L42-44** `extracted_at:
DateTime(tz)` — stamped when L2's extraction stage runs on a bare document; an **idempotency
guard** so the scheduler doesn't re-extract.

### `StgSignal` → `stg_signals` (L47-70)
A competitive/market/technology **event** detected in a document.
**L49-51** `id` — [std autoincrement PK]. **L52** `document_id: FK→stg_documents.id`.
*Crawler-supplied fields:* **L54** `stream` — `competitive|market|technology`. **L55**
`competitor_id`. **L56** `detected_products: JSONB`. **L57** `detected_country`. **L58**
`tech_domain`. **L59** `event_summary: Text` (non-null). **L60-62** `deal_value_raw` /
`deal_value_num: Numeric` / `deal_currency`. **L63** `published_at`.
*L2-computed fields:* **L65** `resolved_competitor_id` — after entity resolution. **L66** `dir`
— `threat|watch|fav` (direction/color). **L67** `lens`. **L68** `tags: JSONB`. **L69**
`dedup_group`. **L70** `proc_status: String default='received' index` — the state-machine
column.

### `StgTender` → `stg_tenders` (L73-94)
A procurement **tender** — the input to `tender_scoring.process_tender` (see `05-services.md`).
**L75-77** `id` — [std autoincrement PK]. **L78** `document_id: FK→stg_documents.id`.
**L79** `source_ref`. **L80** `title: String` (non-null). **L81** `issuer`. **L82** `country`.
**L83** `category_hint` — which category (matched to `ref_kssl_products.category_id` when
scoring). **L84-86** `value_raw` / `value_num: Numeric` / `value_currency` — the tender's
budget as scraped + parsed. **L87** `qty_raw`. **L88** `deadline_date: Date` — bid deadline
(scoring turns this into days-remaining + status). **L89** `requirement_text: Text` — free-text
requirements. **L90** `requirement_fields: JSONB` — structured `[{label,value}]` requirements;
**this is what `parse_requirements` reads to build the spec slots**.
*L2-computed:* **L92** `value_usd: Numeric` — FX-normalized value. **L93** `category_id` — the
resolved category. **L94** `proc_status` default `'received'`.

### `StgPartnership` → `stg_partnerships` (L97-115)
A partnership/deal between a competitor and a partner.
**L99-101** `id` [std autoincrement PK]. **L102** `document_id` FK. **L103** `competitor_id`.
**L104** `partner_name` (non-null). **L105** `partner_id`. **L106** `partner_country`. **L107**
`partner_kind`. **L108** `rel_type`. **L109** `ptype_raw`. **L110** `deal_value_raw`. **L111**
`date_announced: Date`. **L112** `description: Text`. **L113** `detected_lines: JSONB`. **L114**
`kssl_relevance` — how relevant to KSSL. **L115** `proc_status`.

### `StgGeo` → `stg_geo` (L118-135)
A geographic presence/contract fact (competitor X present in country Y with product Z).
**L120-122** `id` [std autoincrement PK]. **L123** `document_id` FK. **L124** `competitor_id`.
**L125** `country`. **L126** `product_name` / **L127** `product_id` / **L128**
`product_category`. **L129** `contract_value_raw`. **L130** `qty_raw`. **L131** `since_year`.
**L132** `stage`. **L133** `note: Text`. **L134** `confidence`. **L135** `proc_status`.

### `StgInnovation` → `stg_innovation` (L138-151)
A technology/innovation item.
**L140-142** `id` [std autoincrement PK]. **L143** `document_id` FK. **L144** `tech_domain`.
**L145** `title` (non-null). **L146** `competitor_id`. **L147** `driver`. **L148**
`maturity_hint`. **L149** `horizon_hint`. **L150** `description: Text`. **L151** `proc_status`.

### `StgAssetAnalysis` → `stg_asset_analysis` (L154-175)
**Multimodal** analysis of a document's captured assets (images/PDFs/screenshots), one row per
analyzed asset. Written by `services/multimodal.py`.
**L155-159** Docstring: `method` records how the row was produced so `/explain` stays honest
about model-vs-rule provenance. **L162-164** `id` [std autoincrement PK]. **L165**
`document_id: FK index`. **L166** `asset_kind` — `image|pdf|screenshot`. **L167**
`asset_index: Integer` — nth asset of its kind. **L168** `storage_path`. **L169** `method` —
`vision_llm|pdf_text`. **L170** `caption: Text`. **L171** `labels: JSONB` — recognised
systems/entities. **L172** `extracted_specs: JSONB` — `[{label,value}]` pulled from a PDF.
**L173** `status` — `ok|empty|error`. **L174** `llm_run_id: Integer` — link to the `llm_runs`
ledger row. **L175** `created_at`.

### `StgCompanyEvent` → `stg_company_events` (L178-191)
A corporate event (M&A, leadership, etc.).
**L180-182** `id` [std autoincrement PK]. **L183** `document_id` FK. **L184** `competitor_id`.
**L185** `event_type`. **L186** `headline` (non-null). **L187** `deal_value_raw`. **L188**
`date_of_event: Date`. **L189** `description: Text`. **L190** `detected_lines: JSONB`. **L191**
`proc_status`.

---

## `models/serving.py` — the client-facing tables (`srv_*`)

**Purpose:** the **only** tables Layer 3 reads. Denormalized and pre-computed — "every value the
UI shows is a literal column here, so the client never computes a score, rank, or color."
**L1-5** Docstring. **L11-15** Imports + `Base`.

### `SrvSignal` → `srv_signals` (L18-41)
"One row = one card in an overview feed." Written by `signal_pipeline.process_signal`.
**L22** `id: BigInteger primary_key` — (note: **mirrors the staging id**, not autoincrement —
the pipeline sets it). **L23** `pillar: String index` — `competitive|market|technology`.
**L24** `dir` — `threat|watch|fav` (**pre-computed color**). **L25** `rank: Integer` —
**pre-sorted**; client just `ORDER BY rank`. **L26** `rank_group`. **L27** `title`. **L28**
`meta`. **L29** `company`. **L30** `lens`. **L31** `sowhat: Text` — the "so what" takeaway.
**L32** `tags: JSONB`. **L33** `ago_display` — human "3d ago" string. **L34** `source_url`.
**L35** `provenance: String default='sourced'` — `sourced|estimate`. **L36** `published_at`.
*Trust spine (Phase 1), all deterministic:* **L38** `confidence: Integer` (0-100). **L39**
`confidence_band` — `high|medium|low`. **L40** `confidence_parts: JSONB` — the decomposition.
**L41** `corroboration: Integer default=1` — count of independent sources (set by
`corroboration.py`).

### `SrvEvidence` → `srv_evidence` (L44-65)
**The evidence chain / XAI backbone.** Every `srv_*` field → the exact source rows that back
it. Written by *every* publish path (rule- and LLM-produced) so explainability is uniform.
**L51** table. **L53-55** `id` [std autoincrement PK]. **L56** `target_kind: String index` —
`signal|tender|partnership|kg_edge|...`. **L57** `target_id: String index` — the serving PK as
text. **L58** `field` — which field this backs (`'sowhat'|'card'|'vulnerability:0'|...`).
**L59** `evidence_id` — the source id (`'doc:doc_8a91'|'sig:412'|...`). **L60** `quote: Text`.
**L61** `source_url`. **L62** `source_tier: Integer`. **L63** `published_at`. **L64**
`method: String default='rule'` — `rule|llm|llm_verified`. **L65** `llm_run_id: BigInteger` —
NULL for rule-produced fields, else the ledger row.

### `SrvSignalDetail` → `srv_signal_details` (L68-82)
Right-panel detail, **1:1 with `srv_signals`** (PK is the signal id, not autoincrement).
**L72** `signal_id: BigInteger primary_key`. **L73** `rank_display`. **L74** `dir`. **L75**
`title`. **L76** `facts: JSONB`. **L77** `what_text: Text` / **L78** `why_text: Text`. **L79**
`lens_reads: JSONB`. **L80** `actions: JSONB`. **L81** `suggest: JSONB`. **L82** `source_url`.

### `SrvTender` → `srv_tenders` (L85-103)
The client-facing tender card. Written by `tender_scoring.process_tender` via `db.merge`.
**L87** `id: BigInteger primary_key` — **same id as the `stg_tenders` row** (merge keyed on it).
**L88** `title`. **L89** `issuer`. **L90** `country`. **L91** `category`. **L92**
`value_display` — the raw string shown. **L93** `value_usd: Numeric` — FX-normalized. **L94**
`qty`. **L95** `deadline_date: Date`. **L96** `dl_days: Integer` — days remaining (**recomputed
daily**). **L97** `req_note: Text`. **L98** `requirements: JSONB` — `[{label,value}]` to render.
**L99** `lean` — `go|maybe|pass` (the LLM verdict). **L100** `lean_text: Text` — the rationale.
**L101** `status` — `open|closing|closed`. **L102** `source_url`. **L103** `provenance`.

### `SrvTenderMatch` → `srv_tender_matches` (L106-116)
**One row per (tender, KSSL product) scored pair.** Written by `process_tender`
(delete-then-insert per tender).
**L108-110** `id` [std autoincrement PK]. **L111** `tender_id: BigInteger index` — FK-ish to
`srv_tenders.id`. **L112** `kssl_product_id`. **L113** `kssl_product_name` (non-null). **L114**
`fit_level` — `high|medium|low`. **L115** `fit_pct: Integer` — the 5-98 score. **L116**
`match_lines: JSONB` — the `["up"/"down", "range 40km meets…"]` reason lines.

### `SrvOverviewMetrics` → `srv_overview_metrics` (L119-125)
Pre-computed metric strip per overview header (client renders zero math). Written by
`metrics.build_overview_metrics`.
**L123** `pillar: String primary_key`. **L124** `generated_at: DateTime(tz)`. **L125**
`metrics: JSONB` — the metric tiles.

### `SrvMatchup` → `srv_matchups` (L128-147)
One KSSL product benchmarked against one competitor product. Recomputed by the matchup engine.
**L132-134** `id` [std autoincrement PK]. **L135** `category: String index`. **L136** `dir` —
`threat|watch|fav` (who leads). **L137** `country`. **L138** `comp_name` (non-null). **L139**
`comp_by`. **L140** `kssl_name` (non-null). **L141** `edge_score: Integer` — 0-100, KSSL's edge.
**L142** `adv_comp: JSONB` / **L143** `adv_kssl: JSONB` — advantage lines each side. **L144**
`verdict: Text`. **L145** `edge_parts: JSONB` — per-spec contribution to the edge score. **L146**
`provenance default='estimate'`. **L147** `verdict_method: String default='rule'` — `rule|llm`.

### `SrvMatchupSpec` → `srv_matchup_specs` (L150-159)
Per-spec comparison rows for a matchup.
**L152-154** `id` [std autoincrement PK]. **L155** `matchup_id: BigInteger index`. **L156**
`spec_label`. **L157** `comp_value` / **L158** `kssl_value`. **L159** `leader` — `comp|kssl|tie`.

### `SrvGeoEntry` → `srv_geo_entries` (L162-178)
Client-facing geo presence rows.
**L164-166** `id` [std autoincrement PK]. **L167** `competitor_id: index` / **L168**
`competitor_name`. **L169** `country: index`. **L170** `product_name`. **L171** `category`.
**L172** `contract_value`. **L173** `since_year`. **L174** `qty`. **L175** `stage`. **L176**
`note: Text`. **L177** `provenance default='sourced'`. **L178** `source_url`.

### `SrvPartnership` → `srv_partnerships` (L181-197)
Client-facing partnership rows.
**L183-185** `id` [std autoincrement PK]. **L186** `competitor_id: index` / **L187**
`competitor_name`. **L188** `partner_name` (non-null). **L189** `partner_kind`. **L190**
`rel_type`. **L191** `country`. **L192** `deal_value`. **L193** `date_announced: Date`. **L194**
`kssl_relevance` — `CORE|ADJACENT|context`. **L195** `meaning: Text`. **L196** `provenance`.
**L197** `source_url`.

### `SrvInnovation` → `srv_innovation` (L200-215)
Client-facing innovation/tech rows.
**L202-204** `id` [std autoincrement PK]. **L205** `tech_domain_id: index`. **L206** `title`
(non-null). **L207** `maturity` — `concept|dev|test|ioc|foc`. **L208** `gap_vs_kssl` —
`ahead|parity|behind`. **L209** `driver`. **L210** `horizon`. **L211** `body: Text`. **L212**
`impact: Text`. **L213** `action: Text`. **L214** `provenance`. **L215** `source_url`.

### `SrvPatent` → `srv_patents` (L218-230)
Patent rows (populated by `patent_sync.py`).
**L220** `id: String primary_key`. **L221** `competitor_id: index` / **L222**
`tech_domain_id: index`. **L223** `jurisdiction`. **L224** `title` (non-null). **L225**
`status` — `granted|pending|filed`. **L226** `filed_date`. **L227** `assignee`. **L228**
`abstract: Text`. **L229** `kssl_relevance`. **L230** `provenance default='estimate'`.

### `SrvCompetitorSynthesis` → `srv_competitor_synthesis` (L233-247)
One synthesized profile per competitor (LLM-generated, `competitor_synthesis.py`).
**L235** `competitor_id: String primary_key`. **L236** `competitor_name`. **L237** `thesis:
Text`. **L238** `strat_pattern: Text` / **L239** `strat_sowhat: Text`. **L240**
`vulnerabilities: JSONB` — `[{title,intel}]`. **L241** `predictions: JSONB`. **L242** `moves:
JSONB`. **L243** `provenance default='estimate'`. **L244** `gaps: JSONB` — declared evidence
gaps. **L245** `confidence: Integer` / **L246** `confidence_band`. **L247** `updated_at`.

### `SrvFieldPattern` → `srv_field_patterns` (L250-260)
Cross-cutting "field pattern" cards (`field_patterns.py`).
**L252-254** `id` [std autoincrement PK]. **L255** `title` (non-null). **L256** `summary: Text`.
**L257** `exceptions: Text`. **L258** `ord: Integer` — display order. **L259** `bottom_line:
Text`. **L260** `provenance default='estimate'`.

---

## `models/graph.py` — knowledge graph (`kg_*`) + graph projections

**Purpose:** a label-property graph in tabular form — a **pure deterministic projection** of
`ref_*`/`srv_*`, fully rebuilt each run (so incremental bugs self-heal). Analytics write back
onto node columns. Written by `graph_builder.py` + `graph_analytics.py`.
**L1-10** Docstring (edge evidence reuses `srv_evidence` with `target_kind='kg_edge'`). **L16-20**
Imports + `Base`.

### `KgNode` → `kg_nodes` (L23-36)
**L25** `id: String primary_key` — `'{kind}:{key}'`, e.g. `'competitor:LT'`. **L26**
`kind: String index` — `competitor|org|product|country|tender|signal|patent`. **L27** `label`.
**L28** `ref_table` / **L29** `ref_id` — lineage back to the source row. **L30** `attrs: JSONB`.
**L31** `provenance default='sourced'` — `sourced|estimate|analyst`.
*Analytics (filled after each rebuild):* **L33** `degree: Integer`. **L34** `betweenness:
Numeric`. **L35** `eigenvector: Numeric`. **L36** `community_id: Integer`.

### `KgEdge` → `kg_edges` (L39-52)
**L41-43** `id` [std autoincrement PK]. **L44** `src_id: index` / **L45** `dst_id: index` —
node ids. **L46** `rel: String index` —
`makes|partners_with|present_in|competes_with|fits|about|filed|issued_in`. **L47**
`rel_subtype default=''` — `jv|license|Contracted|...`. **L48** `weight: Numeric default=1` —
contributing-row count. **L49** `confidence: Integer`. **L50** `provenance`. **L51** `attrs:
JSONB` — `deal_value, stage, fit_pct...`. **L52** `first_seen: DateTime(tz)`.

### `SrvGraphInsight` → `srv_graph_insights` (L55-70)
Hidden-pattern cards, same shape as signal cards (drop into the feed as-is).
**L59-61** `id` [std autoincrement PK]. **L62** `kind: index` —
`shared_partner|broker|predicted_bidder|community`. **L63** `dir default='watch'`. **L64**
`rank: Integer default=999`. **L65** `title`. **L66** `sowhat: Text`. **L67** `entities: JSONB`
— node ids involved. **L68** `metric: Numeric`. **L69** `provenance`. **L70** `computed_at`.

### `SrvAllianceGraph` → `srv_alliance_graph` (L73-81)
One prebuilt node-link payload for the client's network view.
**L77** `id: String primary_key default='latest'` — a single latest-snapshot row. **L78**
`generated_at`. **L79** `nodes: JSONB` / **L80** `edges: JSONB` / **L81** `stats: JSONB`.

---

## `models/llm_ops.py` — the LLM ledger (`llm_runs`)

**Purpose:** cache + idempotency guard + XAI audit trail for **every structured LLM call**. A
cache hit reuses `output` for an identical `(task, input_hash, model)`, so re-processing an
unchanged page costs nothing. Written/read by `services/llm/`.
**L1-7** Docstring. **L13-17** Imports — note it imports **both** `JSONB` (Postgres) and `JSON`
(generic). **L21** `_JSON = JSON().with_variant(JSONB, "postgresql")` — a column type that is
`JSONB` on Postgres but plain `JSON` elsewhere (same Python API); used for the payload columns.

### `LlmRun` → `llm_runs` (L24-43)
**L28-30** `id` [std autoincrement PK] (Postgres BIGSERIAL). **L31** `task: String index` — e.g.
`'classify_signal'`. **L32** `input_hash: String index` — sha256 of task+template+evidence+
params (**the cache key**). **L33** `model`. **L34** `provider`. **L35** `prompt_template_ver
default='v1'`. **L36** `evidence_ids: _JSON`. **L37** `output: _JSON` — the validated result
(what a cache hit returns). **L38** `validator_results: _JSON`. **L39** `status: String` —
`ok|invalid|fallback|error`. **L40** `latency_ms: Integer`. **L41-43** `created_at:
DateTime(tz)` — defaults to `now(utc)` via a lambda (so each row stamps its own insert time).

---

**Next:** `03-contracts.md` — the pydantic request/response schemas that sit between the API and
these tables.
