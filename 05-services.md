# 05 — Services: the processing brain ("vs KSSL" compute)

This is where `stg_*` rows become `srv_*` rows. Every service reads staging/reference, computes
deterministically where it can, calls the LLM only for prose/judgment, and writes serving +
evidence. Grouped by role:

1. **Shared spec logic** — `spec_extract.py`
2. **Flagship pipeline** — `tender_scoring.py` (traced line-by-line)
3. **Document → records** — `extraction.py`
4. **Per-domain pipelines** — `signal_pipeline.py`, `domain_pipeline.py`
5. **Shared helpers** — `entity_resolution.py`, `corroboration.py`, `confidence.py`,
   `evidence.py`, `metrics.py`, `asset_client.py`
6. **Multimodal** — `multimodal.py`
7. **Graph** — `graph_builder.py`, `graph_analytics.py`
8. **Synthesis engines** — `matchup_synthesis.py`, `competitor_synthesis.py`,
   `field_patterns.py`, `patent_sync.py`
9. **Assistant** — `assistant.py`
10. **The LLM stack** — `llm/` (transport, tasks, schemas, validators, cache, stub, legacy)

`services/__init__.py` (**L1-7**) is docstring only: the processing services mirror the service
catalog in `docs/02_DATA_ENGINEERING.md`; LLM-dependent steps go through the `llm` provider so
the whole system runs offline on a deterministic stub.

---

## `spec_extract.py` — spec-slot extraction (shared by scoring + matchups + multimodal)

**Purpose:** map a free-text requirement label (e.g. "Max range") to a normalized **slot**
(`range_km`) with a unit + polarity, and parse `{label,value}` requirement fields into
`{slot: (operator, number)}`. Data-driven: `seed_data/spec_slots.json` extends the builtin table.
**Tables:** none (reads a seed JSON file). **Used by:** `tender_scoring`, matchups, multimodal.

**L18-23** `_BUILTIN_SLOTS` — the fallback slot table: `range_km` (keywords `range`, unit `km`,
polarity `higher_better`), `weight_t` (`weight`/`mass`, `t`, `lower_better`), `calibre_mm`
(`calibre`/`caliber`/`system`/`gun`, `mm`, `match`).
**L26-36** `slot_table()` — `@lru_cache`d: start from the builtin dict, then overlay
`spec_slots.json` from the seed dir if present. A bad seed file is swallowed (`except: pass`) so
scoring never breaks — the builtin table stands.
**L39-44** `slot_for(label)` — lowercase the label, return the first slot whose any keyword is a
substring; else `None`.
**L47-48** `unit_for(slot)` / **L51-52** `polarity_for(slot)` — table lookups with defaults.
**L55-57** `first_number(text)` — regex the first `\d+(.\d+)?` (commas stripped) → float or None.
**L60-66** `required_op(text)` — map wording to a comparison operator: `≥/at least/min/exceed` →
`">="`, `≤/max/under/less than` → `"<="`, else `"=="`.
**L69-77** `parse_requirements(fields)` — **the entry point tender scoring calls.** For each
`{label,value}`, resolve the slot + parse the number; when both present, store
`out[slot] = (required_op(value), value)`. **Input:** `[{label,value}]`. **Output:**
`{slot: (op, number)}`.

---

## `tender_scoring.py` — S-12/S-13 (FLAGSHIP, line-by-line)

**Purpose:** the canonical "new tender → auto-scored vs all KSSL products" flow. Parse a tender's
requirements into slots, score every in-category KSSL product spec-by-spec (deterministic fit%
with up/down reasons), ask the LLM only for the go/maybe/pass verdict, publish `srv_tenders` +
`srv_tender_matches`.
**Reads:** `stg_tenders` (input row), `ref_kssl_products`, `ref_product_specs`, `stg_documents`.
**Writes:** `srv_tenders`, `srv_tender_matches`, and flips the `stg_tenders` row to `published`.
**Called from:** `pipeline/runner.py:62` for each pending tender.

**L8-19** Imports: `datetime` (deadline math), `select`/`Session`, the ORM models
(`RefKsslProduct`, `RefProductSpec`, `SrvTender`, `SrvTenderMatch`, `StgDocument`, `StgTender`),
`LLMProvider`, and the shared spec functions (`parse_requirements`, `polarity_for`, `slot_for`,
`unit_for`).
**L22** `_FX_TO_USD` — a rough display-only FX table (INR/EUR/GBP/USD). Comment flags it as a
placeholder for a real S-04 rates table.
**L25-26** `_slot_for`/`_parse_requirements` — back-compat aliases to the `spec_extract` functions
(the logic moved out to be shared).

**L29-40 `_kssl_specs(db, product_id) -> dict[slot, float]`** — load one KSSL product's specs as
slots.
- **L30-34 QUERY:** `select(RefProductSpec).where(product_side=='kssl' AND product_id==product_id)`.
- **L36-39** for each spec row, map its `spec_label` to a slot; if the slot resolves and
  `value_num` is set, store `specs[slot] = float(value_num)`.
- **Output:** `{slot: numeric_value}` — the product's comparable specs.

**L43-65 `_score_product(reqs, specs, product_name) -> (fit_pct, match_lines)`** — the
deterministic scorer for one product.
- **L45** `score = 55` — baseline "category already matches" (a product only reaches here if it's
  in-category).
- **L47-64** for each required `slot → (op, req_val)`:
  - **L48** `ksv = specs.get(slot)` — the product's value for that slot. **L49** `unit = unit_for`.
  - **L50-51** if the product has no value for that slot, skip it (neither reward nor penalty).
  - **L52-57** `ok` = the requirement is met: operator satisfied (`>=`/`<=`/`==` within 1e-6),
    **or** the slot is `higher_better` and `ksv >= req_val` (a polarity-based pass when no explicit
    operator).
  - **L58** `label = slot.split("_")[0]` — the human prefix (e.g. `range`).
  - **L59-64** met → `score += 14` and an `["up", "range 40km meets the 30km bar"]` line; not met →
    `score -= 8` and a `["down", ...]` line. (`{:g}` trims trailing zeros.)
- **L65** clamp the score to `[5, 98]`. **Output:** `(fit_pct, [[dir, text], ...])`.

**L68-69 `_fit_level(pct)`** — `high` ≥ 80, `medium` ≥ 55, else `low`.
**L72-75 `_value_usd(value_num, currency)`** — multiply by the FX factor (default 1.0 for unknown
currency), rounded; `None` in → `None` out.

**L78-147 `process_tender(db, llm, st)`** — the orchestration.
- **L79** `category_id = st.category_hint` — which category to score against.
- **L80** `reqs = _parse_requirements(st.requirement_fields)` — parse the tender's requirements to
  `{slot:(op,val)}`. **This consumes the JSONB the ingest/extraction/multimodal stages wrote.**
- **L82-84 QUERY:** `select(RefKsslProduct).where(category_id==category_id)` — every KSSL product
  in the tender's category.
- **L86-100** for each product: score it (`_score_product` over `_kssl_specs(db, p.id)`), track
  `best_pct`, and build an in-memory `SrvTenderMatch(tender_id=st.id, kssl_product_id=p.id,
  kssl_product_name=p.name, fit_level=_fit_level(pct), fit_pct=pct, match_lines=lines)`.
- **L101** sort matches by `fit_pct` descending (best first).
- **L103-108** count `high` fits and build a `summary` string (or "No KSSL product in this
  category.").
- **L109** `verdict = llm.tender_verdict(title=..., best_fit_pct=best_pct, match_summary=summary)`
  — **the only LLM call**; returns `{lean, lean_text}` (`go|maybe|pass` + rationale). The prompt is
  told the fit% is computed — cite, don't change it.
- **L112-117** deadline math: `dl_days` = days until `deadline_date`; `status` = `closed`
  (<0) / `closing` (≤7) / `open`.
- **L119** `doc = db.get(StgDocument, st.document_id)` — for the source URL.
- **L120-139 WRITE:** `db.merge(SrvTender(id=st.id, ...))` — upsert the serving card keyed on the
  tender id. Fields: title/issuer/country/category, `value_display=st.value_raw`,
  `value_usd=_value_usd(...)`, `qty`, `deadline_date`, `dl_days`, `req_note=st.requirement_text`,
  `requirements=[{label,value} ...]` (rebuilt from `st.requirement_fields`), `lean`/`lean_text`
  from the verdict, `status`, `source_url`. `merge` = insert-or-update, so re-scoring is idempotent.
- **L140-142 WRITE:** delete all existing `SrvTenderMatch` rows for this tender, then `db.add`
  each new match. (Delete-then-insert keeps the match set exactly current.)
- **L144-146** update the staging row: stamp `value_usd`, `category_id`, and set
  `proc_status="published"` — the state-machine transition that removes it from the pending set.
- **Input:** one `StgTender`. **Output:** one `SrvTender` + N `SrvTenderMatch` rows; the tender
  published. No return value (mutates the session).

---

## `extraction.py` — document → typed records (the L1→pipeline bridge)

**Purpose:** the crawler sends **one bare page bundle**; this stage derives the typed `stg_*`
records from a stored `stg_documents` row. LLM-primary (fast model) with a **regex fallback**, so
ingestion never depends on a model being up. Idempotent via `extracted_at`.
**Reads:** `stg_documents`, `ref_competitors`, `ref_tech_domains`. **Writes:** the six `stg_*`
record tables + stamps `stg_documents.extracted_at`.

**L39-62** Compiled regexes + maps: `_TENDER_PAT`, `_PARTNER_PAT`, `_ACQ_PAT`, `_GEO_PAT`,
`_VALUE_PAT` (currency+amount), `_DEADLINE_PAT` ("closing in N days"), `_REL_TYPE` (keyword→rel),
and `_CATEGORY_KEYWORDS` (artillery/uav/ammunition/small_arms/missiles_ad → trigger words).
**L65-69 `_entities(doc)`** — group `doc.entities_detected` by their `type`.
**L72-77 `_category_hint(text)`** — first category whose keyword appears in the text.
**L80-87 `_tech_domain(db, text)`** — **QUERY** `select(RefTechDomain)`; return the first domain
whose seed `keywords` match the text.

**L90-170 `_regex_extract_document(db, doc)`** — the deterministic fallback. Builds `text` from
title+body, pulls competitor/country/products/partner from entities, finds a value.
- **L107-117** every kept page yields **one signal**: pick `stream` (market if tender, technology
  if a tech-domain match with no competitor, else competitive) and `db.add(StgSignal(...))`.
- **L119-134** if tender-like: parse a deadline, and build `requirement_fields` from the document's
  **tables** (each `{label,value}` row) — this is the raw material tender scoring later parses.
- **L136-146** partnership: if a partner pattern + competitor + partner present, add `StgPartnership`
  with the mapped `rel_type`.
- **L148-158** geo: competitor + country + a geo verb → `StgGeo` with stage Contracted/Offered.
- **L160-168** company event: competitor + acquisition verb → `StgCompanyEvent`.
- **Output:** a per-type `counts` dict.

**L178-246 `_apply_llm_records(db, doc, out, known_ids)`** — map an LLM `ExtractOut` dict onto the
same `Stg*` rows the regex path writes.
- **L187-189** require a valid `stream` + non-empty `summary`, else return `{}` (caller falls back
  to regex).
- **L191-193** `_cid(rec)` — trust a `competitor_id` **only if it's in `known_ids`** (drops
  hallucinated ids).
- **L196-244** add the signal (always) + tender/partnership/geo/event when present. **L205-209**
  bounds the LLM's `deadline_days` to `0..3650` (a giant/negative value would overflow
  `timedelta`).
**L249-254 `_table_fields(doc)`** — the same table→`{label,value}` extraction, shared.

**L257-272 `extract_document(db, doc, llm, known_ids)`** — single-doc wrapper: call
`llm.extract_records(...)`; apply; on empty → regex fallback; stamp `extracted_at`.
**L275-328 `extract_pending(db, llm)`** — the batch entry point (called first in the runner).
- **L283 QUERY:** `select(StgDocument).where(extracted_at IS NULL)` — un-extracted docs.
- **L285-295** for each, check via `select(StgSignal.id/StgTender.id ... limit 1)` whether records
  were already supplied at ingest; if so, just stamp `extracted_at` and skip (mock feeder / tests).
- **L297** `known_ids` = every `ref_competitors.id`.
- **L299-316** **fan out** the LLM extraction over a `ThreadPoolExecutor(max_workers=4)` using a
  **db-less** llm (`worker_llm = llm.with_db(None)`) — because a SQLAlchemy Session isn't
  thread-safe and the db-bound llm's cache/ledger writes would deadlock under 4 threads. Workers
  only do HTTP → dict.
- **L318-327** apply serially on the calling thread: LLM records or regex fallback, stamp
  `extracted_at`, accumulate totals. **Output:** totals dict.

---

## `signal_pipeline.py` — S-07/S-09/S-10 (classify → enrich → publish → rank)

**Purpose:** take a received `stg_signals` row through resolution → classification → enrichment,
write `srv_signals` + `srv_signal_details` + evidence, then rank all cards per pillar.
**Reads:** `stg_signals`, `stg_documents`, `ref_competitors`. **Writes:** `srv_signals`,
`srv_signal_details`, `srv_evidence`.

**L22-23** `_DIR_WEIGHT` (threat 3 / watch 2 / fav 1) + `_GROUP_LABEL` for ranking/display.
**L26-35 `_ago(published_at)`** — "today" / "Nd ago" / "Mon YYYY" relative string.
**L38-112 `process_signal(db, llm, ss, corroboration=1)`** — the per-signal flow.
- **L40-41** load the source document; build `text` = summary + body.
- **L43-46** `resolve_competitor(...)` → `cid`; load the `RefCompetitor`; `company` name; store
  `ss.resolved_competitor_id`.
- **L48-52** `cls = llm.classify_signal(...)` → `{dir, lens, tags}`; stamp them on `ss`. (In the
  task provider, `dir`/`tags` stay deterministic; only `lens` is LLM-picked.)
- **L54-62** build a `facts` list (Company/Domain/Country/Value) from present fields.
- **L64-67** `enr = llm.enrich_signal(...)` → `{sowhat, what_text, why_text, lens_reads, actions,
  suggest}`.
- **L69-71** `meta` = " · "-joined tech_domain/company/value.
- **L73-80** `conf.score(...)` → deterministic `(score, band, parts)` over source tier +
  corroboration + freshness + provenance.
- **L82-91 WRITE:** `db.merge(SrvSignal(id=ss.id, ..., rank=999, ...))` — the feed card (rank is a
  placeholder until `recompute_ranks`).
- **L92-99 WRITE:** `db.merge(SrvSignalDetail(signal_id=ss.id, ...))` — the right-panel detail.
- **L101-111** `write_evidence(...)` — link the source document as evidence for the `card` field
  (method `rule`).
- **L112** `ss.proc_status = "published"`.
**L115-122** `_MIN_AWARE` + `_aware(d)` — normalize naive/aware datetimes so SQLite + Postgres
sort together.
**L125-138 `recompute_ranks(db)`** — S-10: load all `srv_signals`, group by pillar, sort each by
`(dir weight, recency)` descending, and write `rank = 1..n`. Called once after all signals publish.

---

## `domain_pipeline.py` — partnerships / geo / innovation

**Purpose:** for the crawler-fed domains beyond signals/tenders: read a `stg_*` row, resolve the
competitor, apply "vs KSSL" tagging, publish a `srv_*` row **by natural key** (idempotent).
**Reads:** `stg_partnerships`/`stg_geo`/`stg_innovation`, `ref_competitors`, `stg_documents`.
**Writes:** `srv_partnerships`/`srv_geo_entries`/`srv_innovation`.

**L21-32 `_upsert(db, model, key, fields)`** — the idempotency primitive: `select(model)` filtered
by the natural-key columns; if a row exists, update its fields, else insert. So the same
real-world fact reported by a second document updates rather than duplicates.
**L35-39 `_competitor_name`** / **L42-44 `_doc_url`** — small `db.get` lookups.
**L47-76 `process_partnership`** — resolve competitor, derive `relevance` (CORE/ADJACENT/context),
build a KSSL-framed `meaning` string, `_upsert` `SrvPartnership` keyed on
`(competitor_id, partner_name, rel_type)`, publish.
**L79-92 `process_geo`** — resolve competitor, `_upsert` `SrvGeoEntry` keyed on
`(competitor_id, country, product_name)`; provenance `sourced` when confidence high/medium else
`estimate`.
**L95-111 `process_innovation`** — derive `gap_vs_kssl` (behind if a competitor, else parity),
build `impact`/`action` prose, `_upsert` `SrvInnovation` keyed on `(tech_domain, title)`.

---

## `entity_resolution.py` — S-05 (competitor link repair, deterministic, no LLM)

**Purpose:** confirm/repair the crawler's competitor id against `ref_competitors` — exact id match
first, then **word-boundary alias** match over the text.
**L22-31 `_alias_index(db)`** — **QUERY** `select(RefCompetitor)`; build `(alias, id, compiled
word-boundary pattern)` for every name+alias, sorted longest-alias-first (so specific wins).
**L34-39 `_boundary_pattern(alias)`** — for alphanumeric aliases use `(?<![a-z0-9])..(?![a-z0-9])`
so `PEL` doesn't match inside `proPELlant`; multiword/punctuated aliases use plain substring.
**L42-43 `_valid_ids`** — the set of all competitor ids.
**L46-56 `resolve_competitor(db, competitor_id, text)`** — if the given id is valid, keep it; else
scan the alias index against the lowercased text and return the first match; else `None`.

---

## `corroboration.py` — S-06/S-08 (independent-source counting)

**Purpose:** how many **independent** sources back a claim, so a thrice-confirmed award outranks a
lone blog. Two additive layers: a deterministic claim key + an optional embedding merge.
**Reads:** `stg_signals` joined to `stg_documents`. **Writes:** nothing (returns a dict).

**L30-45 `_value_bucket(sig)`** — a currency-agnostic value fingerprint (leading 2 significant
digits), so "₹4,500 cr" and "4500 crore" bucket together; `"?"` when no value.
**L48-62 `_claim_key(sig)`** — deterministic identity `competitor|country|dir|valuebucket`; when
there's no value, the tail is `sig{id}` so unrelated value-less stories don't collapse.
**L65-69 `_cosine`** — plain cosine similarity.
**L72-114 `_embed_merge(rows, sig_key)`** — Phase C: if an embed model is configured, embed one
representative text per key (via `OpenAICompatTransport.embed` on the **separate embed endpoint**),
and union-find-merge keys of the **same competitor** with cosine ≥ 0.86. Any failure → empty remap
(deterministic grouping stands). Only ever raises corroboration.
**L117-140 `corroboration_counts(db)`** — the entry point (called first in the runner).
- **L123-126 QUERY:** `select(StgSignal, StgDocument.source_id).join(...)` — every signal + its
  source id.
- **L128** compute each signal's claim key. **L131-133** apply the optional embedding remap.
- **L135-138** group keys → set of distinct source identifiers (source_id, falling back to
  `doc:<document_id>`).
- **L140** return `{signal.id: len(group)}`. **Output:** the corroboration count per signal
  (passed into `process_signal`).
**L143-165 `demo()`/`__main__`** — DB-less self-check: two wordings of one award share a key; a
different event doesn't.

---

## `confidence.py` — deterministic trust scoring (never LLM)

**Purpose:** `confidence (0-100) = source_tier(≤35) + corroboration(≤25) + freshness(≤25) +
provenance(15)`, decomposed into stored `parts` so the UI can show *why*.
**L15-26** the tuning tables: `_TIER_BASE`/`_TIER_LABEL` (tier 1 best → 35 pts), per-pillar
`_HALF_LIFE` for freshness decay, corroboration/freshness/provenance maxes.
**L29-34 `band(score)`** — ≥70 high / ≥45 medium / else low.
**L40-43 `_freshness_points`** — exponential decay `0.5^(age/half_life)` × 25.
**L46-49 `_corroboration_points`** — 1 source → 0 bonus, saturating at 3.
**L52-99 `score(...)`** — combine the four factors (with tz-normalization and an "unknown date →
neutral" branch), clamp to `[5,95]`, and build the `parts` list of `{factor,label,points,max}`.
**Output:** `(total, band, parts)`. Consumed by `signal_pipeline` (and its `parts` land in
`srv_signals.confidence_parts`).
**L102-132 `demo()`/`__main__`** — self-check asserting the trust ordering the feature depends on
(corroboration+authority beats a lone aggregator; stale scores lower; parts sum to the score).

---

## `evidence.py` — the grounding contract (`srv_evidence` writer)

**Purpose:** stable evidence-ids (eids) + a single `write_evidence` used by **every** publish path,
so `/explain` is uniform. **Writes:** `srv_evidence`.
**L27-33** `doc_eid(id)` → `"doc:<id>"`, `stg_eid(kind,id)` → `"<kind>:<id>"` — the eid scheme
(the docstring L7-13 lists all prefixes: `doc:`, `sig:`/`tender:`/…, `spec:`, `ref:`, `img:`/`att:`,
`agg:`).
**L35-47 `EvidenceItem`** — a dataclass (eid, kind, text, source_url, tier, published_at) with
`clipped()` capping text at 400 chars.
**L49-76 `write_evidence(db, *, target_kind, target_id, items, method, llm_run_id, replace)`** —
if `replace`, `delete` all prior `SrvEvidence` for the target (idempotent), then `db.add` one row
per `(field, EvidenceItem)`. Used by signals, tenders (via graph), matchups, synthesis, patterns,
graph edges, multimodal captions.

---

## `metrics.py` — S-11 overview metric strip

**L19-44 `build_overview_metrics(db)`** — **QUERY** `select(pillar, dir, count).group_by(pillar,
dir)` over `srv_signals`; pivot into per-pillar counts; for each pillar build the metric tiles
(Threats/Watch/Favourable with colors + a filter key) plus a Total, and `db.merge` one
`SrvOverviewMetrics` row per pillar. Client renders zero math.

---

## `asset_client.py` — fetch asset bytes

**Purpose:** get a document's `s3://mallory-raw/...` blob, either straight from MinIO (when
`minio_endpoint` set) or by proxying the crawler's `GET /artifact`.
**L17-26 `_client()`** — lazily build a cached `minio.Minio` client. **L29-31 `_object_key`** —
strip scheme+bucket from the URI. **L34-54 `fetch_asset(storage_path)`** — MinIO `get_object` when
configured, else an `httpx` GET to the crawler; raises on missing/unreachable (callers handle).
Used by `serving.asset_proxy` and `multimodal._data_uri`.

---

## `multimodal.py` — Phase B (image captions + PDF spec extraction)

**Purpose:** turn captured assets into intel — images → vision caption + product-label match; PDFs
→ spec rows merged into the owning tender's `requirement_fields` (so tender scoring picks them up
with zero scoring changes). Opt-in (`/ops/analyze-assets`) because the vision model must swap into
VRAM. Idempotent per asset (a `stg_asset_analysis` row exists).
**Reads:** `stg_documents`, `ref_kssl_products`, `ref_competitor_products`, `stg_tenders`.
**Writes:** `stg_asset_analysis`, `srv_evidence`, and mutates `stg_tenders.requirement_fields`.

**L37-45 `_data_uri`** — fetch bytes via `asset_client`, base64-wrap as a `data:` URI for the
vision model. **L48-59 `_product_index`** — lowercased product name/alias → product id (KSSL +
competitor). **L62-71 `_match_labels`** — map vision labels to known product ids.
**L74-79 `_already`** — the set of `{kind}:{index}` already analyzed for a document (idempotency).
**L82-150 `analyze_document_assets`** — for each un-analyzed image: caption via
`llm.caption_image`, match labels, write a `StgAssetAnalysis` row (+ evidence when captioned). For
each un-analyzed PDF attachment with extracted text: `llm.extract_specs`, write a `StgAssetAnalysis`
row, and **merge the specs into the owning tender's `requirement_fields`** (dedup by
`(label,value)`); if that tender was already `published`, flip it back to `received` so it re-scores
with the new specs.
**L153-175 `analyze_pending_assets`** — iterate documents that carry assets and aren't fully
analyzed; a clean no-op when there are no assets or the vision model is disabled. Commits, returns
totals.

---

## `graph_builder.py` — project ref_*/srv_* → kg_nodes/kg_edges (deterministic)

**Purpose:** full idempotent rebuild of the knowledge graph — wipe and re-derive everything, so the
graph is always reproducible. Every edge carries provenance/confidence and (where meaningful)
`srv_evidence` links (`target_kind='kg_edge'`).
**Reads:** `ref_competitors`, `ref_kssl_products`, `ref_competitor_products`, `ref_matchups`, and
`srv_partnerships`/`srv_geo_entries`/`srv_tenders`/`srv_tender_matches`/`srv_signals`/`srv_patents`.
**Writes:** `kg_nodes`, `kg_edges`, `srv_evidence` (kg_edge).

**L30** `_CONF_BY_PROV` — default edge confidence by provenance. **L33-34 `_slug`** — id-safe slug.
**L37-64 `_Builder`** — accumulates unique nodes (`node()` dedups by `{kind}:{key}`) and adds edges
(`edge()` sets confidence from provenance, flushes to get an id, then writes edge evidence).
**L67-175 `rebuild_graph(db)`** — the projection:
- **L69-72** wipe kg-edge evidence, all edges, all nodes.
- **L77-90** competitor nodes (incl. the KSSL anchor) + product nodes, with `makes` edges
  (anchor→KSSL products, competitor→competitor products).
- **L93-105** partnerships → `competitor -[partners_with:{rel_type}]-> org` (with evidence).
- **L108-121** geo → `competitor -[present_in:{stage}]-> country` (with evidence).
- **L124-133** matchups → `kssl product -[competes_with]-> competitor product`.
- **L136-148** tenders → `tender -[issued_in]-> country` and `kssl product -[fits:{fit_pct}]->
  tender`.
- **L150-161** signals → `signal -[about]-> competitor` (confidence from the signal).
- **L163-169** patents → `competitor -[filed]-> patent`.
- **L171-175** persist nodes, flush, return `{nodes, edges}` counts.

---

## `graph_analytics.py` — hidden-pattern miners (NetworkX, deterministic)

**Purpose:** four analyses over `kg_*` → insight cards (`srv_graph_insights`) + the prebuilt
alliance payload (`srv_alliance_graph`); also writes centrality/community back onto `kg_nodes`.
**L25-35 `_alliance_nx(db)`** — build an undirected competitor↔org NetworkX graph over
`partners_with` edges.
**L38-151 `run_analytics(db)`** — call after `rebuild_graph`:
1. **L46-59 shared partners** — an org neighboring ≥2 competitors → a `shared_partner` insight.
2. **L61-74 communities** — Louvain (seed=42) → write `community_id` onto nodes; record blocs.
3. **L76-97 centrality** — degree/betweenness/eigenvector onto nodes; the top org broker → a
   `broker` insight.
4. **L99-127 predicted bidders** — a competitor that `makes` an in-category product **and** is
   `present_in` the tender's country → a `predicted_bidder` (threat) insight.
- **L129-133** rank insights (threats first, then metric). **L135-151** build + `db.merge` the
  `srv_alliance_graph` "latest" payload; return `{insights, ...stats}`.

---

## `matchup_synthesis.py` — S-22 (recompute srv_matchups from ref_matchups)

**Purpose:** deterministic per-spec leader + weighted `edge_score` (highlight ×2); LLM writes only
the verdict prose. Serving rows fully replaced each recompute.
**Reads:** `ref_matchups`. **Writes:** `srv_matchups`, `srv_matchup_specs`, `srv_evidence`.
**L19-27 `_leader(spec)`** — numeric compare by polarity (`better=="low"` → lower wins) else tie.
**L29-48 `_compute(specs)`** — sum weighted points per side, build `edge_parts`, compute
`edge = clamp(50 + 12*(kssl-comp), 5, 95)` and `dir` (fav/threat/watch). **L51-57
`_template_verdict`** — the rule fallback verdict.
**L60-102 `recompute_all(db, llm)`** — delete existing `srv_matchup*`; for each ref matchup:
compute, optionally overwrite the verdict via `llm.matchup_verdict` (method `llm` vs `rule`),
insert `SrvMatchup` + per-spec `SrvMatchupSpec` rows, and `write_evidence` linking the curated ref
row. Returns the row count.

---

## `competitor_synthesis.py` — S-23 (the analyst brain, per competitor)

**Purpose:** GATHER published srv rows as cited evidence → SYNTHESIZE (one deep-model structured
call over evidence, never raw docs) → VERIFY (schema + **every vulnerability must cite** real
evidence) → PUBLISH. Fail-safe: a thin/bad generation never overwrites an existing row.
**Reads:** `ref_competitors`, `ref_kssl_products`, `srv_signals`/`srv_partnerships`/
`srv_geo_entries`. **Writes:** `srv_competitor_synthesis`, `srv_evidence`.
**L35-59 `_gather`** — collect up to 60 `EvidenceItem`s from the competitor's signals (best
confidence first), partnerships, geo. **L62-63 `_render`** — evidence as `[eid] text` lines.
**L66-83 `_exemplar`** — a seed entry (of a *different* competitor) that teaches analytical voice +
the cite mechanic. **L86-88 `_anchor_frame`** — the KSSL product list. **L91-98
`_synthesis_confidence`** — confidence = mean confidence of the cited signal rows.
**L101-170 `synthesize_competitor`** — skip anchor/unknown; require ≥3 evidence items (else keep
existing); call `llm.synthesize_competitor(...)`; **drop any vulnerability whose cites aren't in the
evidence pack** (this is what stops parametric-knowledge claims laundering into "sourced" intel);
if none survive, keep the existing row; else `db.merge` the synthesis (provenance `sourced`,
confidence from cites) and `write_evidence` per field. **L173-175 `synthesize_all`** — run it for
every non-anchor competitor.

---

## `field_patterns.py` — S-24 (cross-field patterns)

**Purpose:** deterministic aggregates (shared-partner pile-ups, contested countries, licensing
concentration) — each a citable `agg:` unit — then optional LLM narrative; only `sourced` rows feed
the aggregates. **Reads:** `srv_partnerships`/`srv_geo_entries`/`srv_competitor_synthesis`.
**Writes:** `srv_field_patterns`, `srv_evidence`.
**L26-35 `Aggregate`** — dataclass (eid, title, summary, member_eids) with `item()` → EvidenceItem.
**L42-90 `compute_aggregates`** — three aggregate families over sourced rows.
**L93-134 `refresh_field_patterns`** — if no sourced aggregates, keep seed rows; else optionally
LLM-narrate (`llm.field_patterns`) or fall back to "the aggregates ARE the patterns"; replace
`srv_field_patterns`, write per-pattern evidence.

---

## `patent_sync.py` — real patents → srv_patents

**Purpose:** replace the seed patent fiction with real filings. Three sources tried in order
(SerpApi → USPTO ODP → keyless Google Patents), normalized to one shape; classify tech domain
against `ref_tech_domains`; upsert `SrvPatent` (provenance `sourced`). Estimate rows are dropped
only once real data arrives (so a total failure leaves the seed sample).
**L50-62** `_jurisdiction` / `_status` from the publication number. **L65-90 `_fetch_google`**
(keyless XHR, backs off on 503). **L93-127 `_fetch_uspto`** (POST search with `X-API-KEY`).
**L129-135 `_first_assignee`** — defensive field pulling. **L138-145 `_fetch`** — the source
dispatch. **L148-171 `_fetch_serpapi`**. **L174-179 `_tech_domain`** — same domain classifier.
**L182-239 `sync_patents`** — for each competitor with a known assignee portfolio (`_ASSIGNEE`),
fetch recent filings, drop estimates on first real data, `db.merge` each `SrvPatent`, rate-limit
between competitors. Returns counts. (Invoked by the `scripts/sync_patents.py` CLI, not the main
pipeline.)

---

## `assistant.py` — S-26 chat + S-25 CEO report (grounded over serving)

**Purpose:** Mallory answers ONLY from the scoped serving rows for the panel the user is in; the
CEO report composes a cross-pillar brief from top serving rows. **Reads:** the `srv_*` tables.
**Writes:** none.
**L26-30** `SYSTEM` — the grounding system prompt ("answer ONLY from the provided context, framed
for KSSL").
**L33-88** context builders, one per panel: `_signal_context` (detail+card), `_tender_context`
(tender + its matches), `_competitor_context` (synthesis + partnerships), `_overview_context` (top
8 signals by rank). Each returns `(context_text, sources)`.
**L91-103 `answer(db, llm, req)`** — pick the context builder by `req.panel_context`+`entity_id`,
call `llm.chat(system, context, message)`, return `MalloryResponse(answer, scope, sources)`.
**L106-137 `ceo_report(db, llm, focus)`** — gather top threats/go-tenders/innovation, build a
context, ask the LLM for a 3-sentence exec summary, assemble the section list, return
`ReportResponse`.

---

## The LLM stack — `llm/`

**Architecture:** `transport` (the only HTTP code) → `tasks` (prompts + validation + fallback) →
`schemas`/`validators`/`cache`. A `stub` gives a deterministic offline path; `providers_legacy`
keeps the old Anthropic/OpenRouter providers working. **Table:** `llm_runs` (cache + ledger).

### `llm/__init__.py` — public API
**L26-48** `LLMProvider` — a `Protocol` declaring the eight methods every provider implements
(classify_signal, enrich_signal, tender_verdict, chat, extract_records, caption_image,
extract_specs — synthesis methods are duck-typed via `getattr`).
**L51-66 `get_llm(settings, *, db)`** — resolve the provider from `settings.llm_provider`:
`ollama` → `OllamaTasksProvider(build_transport, settings, db)`; `openrouter` with a key → the task
provider when a `db` is present (structured output + ledger) else the legacy provider; `anthropic`
→ the legacy Anthropic provider; else the **stub**. Pass `db` to enable cache + ledger.

### `llm/transport.py` — the only per-provider HTTP
**L17-27 `ChatRequest`** — a provider-agnostic request dataclass (system/user/model, json_mode,
json_schema, images, max_tokens, temperature, num_ctx).
**L30-32 `ChatTransport`** — Protocol: `complete(req) -> str|None`, `embed(texts) -> vectors|None`.
**L35-46 `NullTransport`** — every call returns `None` (the `stub` path: no network).
**L48-134 `OpenAICompatTransport`** — serves local Ollama, the remote farm, and OpenRouter (all
speak OpenAI `/v1/chat/completions`).
- **L70-119 `complete`** — build the messages body (multimodal content-parts when `images` set),
  set `response_format` to `json_schema` or `json_object` when asked; **on HTTP 400 for structured
  output, retry in plain mode** with a "respond with JSON only" nudge (task-side brace-extraction
  recovers it); any exception → `None` (nothing raises into the pipeline).
- **L121-134 `embed`** — POST `/embeddings`; used by corroboration's semantic merge.
**L137-152 `build_transport(settings)`** — pick `OpenAICompatTransport` for ollama/openrouter with
a key, else `NullTransport`.

### `llm/tasks.py` — the task layer (provider-agnostic domain methods)
**Flow (docstring L3-9):** cache lookup → transport.complete(schema) → parse+validate (one retry)
→ deterministic validators → write the `llm_runs` ledger → any failure falls back deterministically.
**Numbers/rankings are never model-computed** — this layer owns prose and judgment only.
**L29-31 `norm_cite`** — normalize a model-emitted eid (`"[SIG:1]"` → `"sig:1"`).
**L34-45 `_extract_json`** — parse JSON, tolerating prose around the object (brace extraction).
**L48-124 `OllamaTasksProvider`** + **`_run_structured`** — the core: hash the input
(`cache.input_hash`), check the cache, call the transport, parse+validate against the pydantic
schema (with one error-fed retry), run `validators.numbers_grounded` + the task's `extra_validate`,
record the ledger row (`ok`/`invalid`/`error`/`fallback`), and return the parsed dict or the
fallback. **This is where every structured LLM call is grounded, cached, and audited.**
**L127-347** the eight+ domain methods, each a thin `_run_structured` call with its prompt, schema,
`evidence_text` (what numbers must be grounded against), `extra_validate`, and deterministic
`fallback`: `classify_signal`/`_lens`, `extract_records` (+ `_valid_competitor_ids` drops
hallucinated ids), `caption_image`, `extract_specs`, `enrich_signal`, `tender_verdict`,
`chat`, and the synthesis trio (`synthesize_competitor`, `matchup_verdict`, `field_patterns`, each
using `_cites_valid` to reject uncited claims). **L350-361 `_flatten`** — collect leaf values for
the numbers-grounded scan.

### `llm/schemas.py` — pydantic output schemas + JSON-schema exports
**L13-143** the output models with field constraints — e.g. `ClassifyOut.dir` must match
`^(threat|watch|fav)$`, `TenderVerdictOut.lean` match `^(go|maybe|pass)$`, `EnrichOut.sowhat`
`max_length=400`, `ExtractOut` (signal always + optional tender/partnership/geo/event),
`SynthesisOut` (vulnerabilities `min_length=1`, `cites` `min_length=1`), etc. These enforce the
shape before anything reaches the pipeline.
**L148-163** `_schema()` + the `*_SCHEMA` exports — the JSON Schemas handed to the transport's
`response_format` (title noise stripped).

### `llm/validators.py` — deterministic output guardrails
**L12-16** `_NUM_UNIT` — regex for number-with-unit tokens (₹4,500 cr, 155mm, 52 km, 12%…).
**L23-30 `numbers_grounded(output, evidence)`** — **every number-with-unit in the output must
appear in the evidence** (comma/space-insensitive) — the anti-hallucination check.
**L33-36 `length_bounds`** / **L39-42 `enum_valid`** — cheap length + enum checks. All return a
problem-string list (empty = clean); they never raise.

### `llm/cache.py` — the `llm_runs` cache + ledger
**L18-23 `input_hash`** — sha256 of a canonical `{task, model, template, payload}` — the cache key.
**L26-41 `lookup`** — return the `output` of the latest `ok` run for `(task, model, input_hash)`;
uses `no_autoflush` so the read doesn't force-flush the caller's half-built rows; `None`/error → miss.
**L44-61 `record`** — `db.add` an `LlmRun` (no flush mid-pipeline, so a ledger failure never wipes
the caller's work). Both no-op when `db is None` (stub/CI stays DB-free).

### `llm/stub.py` — the deterministic offline provider
**L22-113 `StubLLMProvider`** — rule-based, KSSL-framed, fully deterministic. `classify_signal`
(keyword→dir/lens/tags), `enrich_signal` (templated so-what/actions), `tender_verdict`
(≥80 go / ≥55 maybe / else pass — the same thresholds the fit_level uses), and `chat` (surfaces the
scoped context honestly). `extract_records`/`caption_image`/`extract_specs` return `{}` so callers
use their deterministic fallbacks. **This is the source of truth for `LLM_PROVIDER=stub` and the
per-task fallback every real provider degrades to.**

### `llm/providers_legacy.py` — the pre-split Anthropic/OpenRouter providers
**L16-80 `AnthropicLLMProvider`** / **L83-159 `OpenRouterProvider`** — the original JSON-over-HTTP
providers, kept unchanged for back-compat. Each implements classify/enrich/verdict/chat and falls
back to the stub on any error. New capability goes through `tasks`, not here.

---

**Next:** `06-pipeline.md` — how `runner.process_pending` sequences all of the above.
