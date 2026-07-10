# 03 — Contracts: the typed API boundaries

`contracts/` holds the **pydantic** models that validate HTTP request bodies and shape response
JSON. They are *not* database tables — they're the wire format. FastAPI validates every request
body against these, rejecting malformed input with **HTTP 422 before it touches the DB**.

- `ingest.py` — **Interface A** (L1→L2): what the crawler must POST. Mirrors the `stg_*` columns
  but "crawler-supplied fields only" (no `proc_status`, no L2-computed columns).
- `serving.py` — **Interface B** (L2→L3): the DTOs the client receives. Mirror the `srv_*` tables
  (already scored/ranked). `from_attributes=True` lets them be built straight from ORM rows.

Because these mirror tables already documented in `02-models.md`, this doc annotates the
**differences that matter** per field: `Literal[...]` enums (accepted values), which fields are
**required** (no default) vs optional, and defaults — not re-explaining columns already covered.

---

## `contracts/__init__.py`
**L1-5** Docstring only: `ingest` = L1→L2 contract, `serving` = L2→L3 DTOs.

---

## `contracts/ingest.py` — what the crawler POSTs

**L1-6** Docstring: single source of truth for the crawler's payload; field names/enums mirror
`docs/01_CRAWLER_CONTRACT.md`. **L10-13** Imports: `datetime`, `typing.Literal` (for closed enums),
pydantic `BaseModel`/`Field`.

### Document sub-objects (nested inside a document)
- **`ImageIn` (L18-24)** — `url` (required) + optional `storage_path`, `caption`, `role`
  (`product|event|chart|person|map|other` — a comment, not enforced), `width`, `height`.
- **`AttachmentIn` (L27-31)** — `url` + optional `storage_path`, `type` (`pdf|xlsx|docx`),
  `extracted_text` (text L1 already pulled from a PDF).
- **`ScreenshotIn` (L34-36)** — `storage_path` (required) + `captured_at`.
- **`TableIn` (L39-41)** — optional `title` + `rows: list[dict]` (default empty via
  `Field(default_factory=list)` — the pydantic-safe way to default a mutable).
- **`EntityDetectedIn` (L44-48)** — `surface` (the text as it appeared, required) +
  `resolved_id`, `type` (`competitor|product|country|partner|unknown_company`), `confidence`.

### `DocumentIn` (L51-72) — the source page
**L52** Docstring: one per kept URL; `main_text` is required because L2 runs NLP on it.
**Required fields (no default):** `url`, `content_hash`, `fetched_at`, `source_id`, `title`,
`main_text`. **Optional:** `published_at`, `source_tier`, `author`,
`date_precision: Literal["exact","approx","unknown"]`, `language`,
`access: Literal["open","paywalled","partial"]`, `main_text_en`, `summary`, and the four nested
lists (`images`, `attachments`, `tables`, `entities_detected`, each defaulting to `[]`) plus a
single optional `screenshot`. These map 1:1 to `stg_documents` columns.

### The six typed records (crawler-supplied fields only)
Each mirrors a `stg_*` table minus the L2-computed columns. `document_id` is optional here
because the page-envelope path supplies it after upserting the document.

- **`CompetitiveSignalIn` (L78-89)** — required: `stream:
  Literal["competitive","market","technology"]`, `event_summary`. Optional: `competitor_id`,
  `detected_products` (list), `detected_country`, `tech_domain`, `deal_value_*`, `published_at`.
  → `stg_signals`.
- **`ReqFieldIn` (L92-94)** — `{label, value}`, both required. This is the shape of a single
  requirement; a tender carries a list of them, and `spec_extract.parse_requirements` reads
  exactly this to build scoring slots.
- **`TenderIn` (L97-110)** — required: `title`. Optional everything else incl.
  `category_hint`, `value_*`, `deadline_date: dt.date`, `requirement_text`, and
  `requirement_fields: list[ReqFieldIn]`. → `stg_tenders`.
- **`PartnershipIn` (L113-125)** — required: `partner_name`. Enum `rel_type:
  Literal["jv","mou","license","supply","customer","investment"]`. → `stg_partnerships`.
- **`GeoFootprintIn` (L128-140)** — all optional; enums `stage:
  Literal["Offered","Trials","Contracted","Delivered"]`, `confidence:
  Literal["high","medium","low"]`. → `stg_geo`.
- **`InnovationIn` (L143-151)** — required: `title`. Enum `maturity_hint:
  Literal["concept","dev","test","ioc","foc"]`. → `stg_innovation`.
- **`CompanyEventIn` (L154-164)** — required: `headline`. Enum `event_type:
  Literal["acquisition","financial","leadership","contract_win","product_launch"]`. →
  `stg_company_events`.

### `PageEnvelopeIn` (L170-177) — the recommended one-shot payload
One `document: DocumentIn` (required) + six lists of the typed records (each default `[]`). The
crawler posts this to `/ingest/v1/page` to write a document and all its records atomically.

---

## `contracts/serving.py` — the DTOs the client receives

**L1-4** Docstring: these mirror `srv_*`; everything is already scored/ranked/"vs KSSL".
**L10** `T = TypeVar("T")` — for the generic `Page[T]`. **L14** declares it.
**L17-18** `class ORMModel(BaseModel)` with `model_config = ConfigDict(from_attributes=True)` —
**the key line**: subclasses can be built directly from an ORM row via
`SomeDTO.model_validate(orm_row)` (attribute access, not dict). Every response DTO subclasses this.

DTOs (each mirrors the same-named `Srv*` table; only the notable bits called out):
- **`SignalCard` (L21-39)** — the feed card. Mirrors `srv_signals` client-facing columns; note
  defaults `provenance="sourced"`, `corroboration=1` (so a row without them still validates).
- **`SignalDetail` (L42-53)** — mirrors `srv_signal_details` (keyed by `signal_id`).
- **`TenderMatch` (L56-61)** — one scored KSSL-product row; mirrors `srv_tender_matches` minus
  ids the client doesn't need.
- **`TenderCard` (L64-81)** — mirrors `srv_tenders`, **plus** `matches: list[TenderMatch] = []`.
  This nested list is **not a DB column** — the serving API fills it by a second query (see
  `_tender_with_matches` in `04-api.md`). This is how one HTTP call returns tender + all its
  product matches.
- **`OverviewMetrics` (L84-87)** — mirrors `srv_overview_metrics`.
- **`Page(BaseModel, Generic[T]) (L90-94)`** — pagination envelope: `items: list[T]`, `page`,
  `size`, `total`. Used as `Page[SignalCard]`.
- **`MatchupSpec` (L97-101)** / **`MatchupCard` (L104-116)** — mirror `srv_matchup_specs` /
  `srv_matchups`; `MatchupCard.specs: list[MatchupSpec] = []` is again filled by a second query.
- **`GeoEntry` (L119-132)**, **`PartnershipCard` (L135-148)**, **`InnovationCard` (L151-163)**,
  **`PatentCard` (L166-177)** — mirror their `srv_*` tables directly.
- **`CompetitorSynthesisDTO` (L180-188)** — a **subset** of `srv_competitor_synthesis` (drops
  `strat_pattern`, `gaps`, `confidence*` — the client card doesn't show them).
- **`FieldPatternDTO` (L191-196)** — subset of `srv_field_patterns` (drops `bottom_line`).

### Assistant + report DTOs (request/response only, no table)
- **`MalloryRequest` (L199-202)** — chat input: `message`, `panel_context`
  (`overview|signal|tender|matchup|competitor`, default `overview`), optional `entity_id`.
- **`MalloryResponse` (L205-208)** — `answer`, `scope`, `sources: list[str]`.
- **`ReportRequest` (L211-212)** / **`ReportResponse` (L215-218)** — CEO report in/out
  (`sections: [{heading, body}]`).

### Explainability DTOs ("why this?")
- **`EvidenceRef` (L224-230)** — one evidence item: `eid`, `quote`, `source_url`, `source_tier`,
  `published_at`, `method` (`rule|llm|llm_verified`, default `rule`). Mirrors a `srv_evidence` row.
- **`FieldExplanation` (L233-236)** — one field + `method` + its `evidence: list[EvidenceRef]`.
- **`ExplainResponse` (L239-247)** — the `/explain` payload: `target_kind`, `target_id`,
  `provenance`, `confidence`/`confidence_band`/`confidence_parts`, `evidence_count`, and
  `fields: list[FieldExplanation]`. Assembled by the `explain` endpoint from `srv_evidence` +
  the owning `srv_*` row.

**Takeaway:** ingest contracts are a *validation gate* in front of `stg_*`; serving contracts are
a *projection* of `srv_*` (sometimes a subset, sometimes with a nested list the API joins in).
