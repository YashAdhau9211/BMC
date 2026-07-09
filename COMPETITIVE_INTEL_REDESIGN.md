# Competitive Player Intelligence — Redesign Strategy

**Status:** design & roadmap only. No code, schema, or prompt changes are made by this document.
**Scope:** the L2 **Competitive Player** intelligence layer only (Movement, Positioning, Partnerships,
Geography). Every other L2 module is out of scope.
**Anchor:** everything is computed *vs KSSL* (Kalyani Strategic Systems Ltd).

---

## 0. Context — why this redesign

The Competitive Player layer today produces outputs that are **generic, inconsistent, weakly reasoned,
and evidence-poor**. The same news can be labelled differently on different runs; a "Threat" rarely
says *why* it threatens KSSL or *which* products are involved; partnerships all render the same
sentence; geography is a pile of disconnected dots.

Root cause (verified in code, §2): the system's central judgement — Threat vs Watch — is **not made
by the AI at all**. It is made by ~8 keywords scanning one summary line, with **no test of whether the
event affects KSSL**. The AI only relabels a cosmetic tag and writes prose to justify a verdict it did
not make.

**The redesign's single principle:** **deterministic-first — agents read, rules decide.**
The AI extracts *facts* (its existing job); a transparent set of **rules** turns facts into verdicts,
scores, and rankings; every conclusion carries **traceable evidence**; and when evidence is missing,
the system **says so** instead of guessing. Same input → same output, always.

**Hard constraints honoured throughout:** pipeline architecture unchanged (extraction → process →
serve); no prompt rewrites; no schema changes (existing columns / JSONB / seed-data absorb everything;
the one genuine schema gap is flagged as a deferred decision). Numbers and rankings are never
model-computed.

**Decisions already taken by the product owner** (baked in below): keep single-source Threats with an
`unconfirmed` tag (not demote); KSSL market list authored in seed data; auto-paired product
comparisons publish as `estimate`; radar/EW size-classing via spec-derived bands; runtime agents limited
to a verifier + cold-path helpers with verdicts staying deterministic; build via parallel per-phase agents.

---

## 1. How the Competitive Player pipeline works today

A crawled page flows: **Ingest → (60s scheduler) process_pending → serve**. Within that, the competitive
records are produced as follows (all paths file:line-verified):

1. **Extraction** — `services/extraction.py`. One page → one `StgSignal` (+ optional partnership/geo).
   LLM-primary (`extract_records`) with a regex fallback; offline ⇒ pure regex. Writes
   `stream="competitive"`, `event_summary`, `competitor_id`, `detected_products`, `detected_country`,
   `deal_value_raw`. The regex path hard-codes `event_type="acquisition"`; there is no event taxonomy.
2. **Signal processing** — `services/signal_pipeline.py::process_signal`. Order: resolve competitor →
   classify (`dir/tags/lens`) → build facts → enrich (prose) → score confidence → publish
   `SrvSignal` + `SrvSignalDetail` + evidence → mark published.
3. **Classification** — `services/llm/tasks.py::classify_signal` **delegates `dir` + `tags` entirely to
   the deterministic stub** (`services/llm/stub.py`) and only asks the LLM for the cosmetic `lens` label.
4. **Confidence** — `services/confidence.py`: `source_tier(≤35) + corroboration(≤25) + freshness(≤25) +
   provenance(15)`, clamp [5,95]. For signals `provenance` is hard-coded `"sourced"` (always +15).
5. **Rank** — `services/signal_pipeline.py::recompute_ranks`: sort by `dir` weight (threat 3 / watch 2 /
   fav 1) then recency. Confidence does not affect order.
6. **Partnerships / Geo** — `services/domain_pipeline.py`: fully templated, the `_llm` argument is
   received and discarded. Last-writer-wins upsert.
7. **Positioning** — `services/matchup_synthesis.py::recompute_all`: reads the **8 hand-curated**
   `ref_matchups` only; no dynamic pairing.

### The live classifier (the actual decision-maker), verbatim behaviour
`services/llm/stub.py:11-42` — precedence: contains a "loss" word → good-news; market + opening word →
watch; contains a "win/award/contract/order" word → **threat**; else the competitor's static default.
It scans `event_summary` **only**, with substring matching, and performs **no KSSL-relevance test**.

---

## 2. Weaknesses in the current reasoning

### Movement (classification)
1. **No KSSL-relevance test exists anywhere.** `stub.py:31-32`: any competitor + any of 8 keywords =
   Threat. A NORINCO naval win in Argentina scores identically to an L&T 155mm order in India.
2. **The "LLM classification" is cosmetic.** `tasks.py:127-136`: the model only relabels `lens`; the
   verdict is 8 keywords over one line.
3. **Substring keyword matching.** `stub.py:27,31`: "win" ⊂ "Darwin", "order" ⊂ "border",
   "grounded" ⊂ "well-grounded".
4. **Keyword order effects.** Loss-words are checked first — "wins ₹2,000cr order despite delay" → good-news.
5. **Identity prior decides keyword-free events.** `stub.py:34`: the same neutral text is Threat for one
   competitor and Watch for another, based on a seeded default, not the event.
6. **No rationale recorded.** `SrvSignal`/`SrvSignalDetail` carry no why-of-label, no gate, no affected
   products; `why_text` is prose told "direction: {dir}" and asked to justify it.
7. **Extracted products are dead data.** `StgSignal.detected_products` is written at `extraction.py:113,198`
   and **read by nothing** — the one field that could establish product relevance is discarded.
8. **Enrichment collapse.** `tasks.py:244-246` grounds enrich only on `event_summary + facts`;
   `validators.py:23-30` rejects any article number beyond that → output falls back to **3 fixed template
   sentences** (`stub.py:44-74`), which is why detail panels read generic.
9. **Provenance is a constant.** `signal_pipeline.py:78`: every signal gets `"sourced"` + 15 points — dead weight.
10. **Confidence never affects order.** A thrice-corroborated tier-1 threat and a lone tier-4 rumour
    interleave by date alone.
11. **Corroboration keys on the label and runs before labelling.** `corroboration.py:58-62` includes
    `dir`; `runner.py:50` computes counts while pending rows have `dir=None` → duplicates never group on
    first pass (systematic undercount).
12. **Misattribution risk.** Resolution scans the whole document; a bystander company mid-article can
    become the classified actor.
13. **No event taxonomy.** Factory expansion, CSR, and a contract win are indistinguishable to the classifier.
14. **Three labels violate the two-label doctrine.** `fav` is live vocabulary across stub/schemas/metrics/rank.

### Positioning
- **P1** Missing specs silently score as "tie" — absent data looks like measured parity
  (`matchup_synthesis.py:23-24`).
- **P2** Frozen at 8 curated pairs; ~89 catalogued competitor products can never surface (`matchup_synthesis.py:62`).
- **P3** `RefMatchup.comp_name` is a free string, not a link — the graph mints a duplicate product node.
- **P4** `RefProductSpec.product_side='competitor'` is defined but **never populated** (loader hard-codes
  `"kssl"`); dynamic comparison starves.
- **P5** 3 spec slots only, no `spec_slots.json`, 15 KSSL rows / 3 labels / artillery only; **no unit
  conversion** (18,000 kg vs 18 t compare as 18000 vs 18); calibre keywords "system"/"gun" mis-map weight lines.
- **P6** No product entity resolution at all; `detected_products` unused, multimodal uses unsafe substring.
- **P7** Extracted PDF specs feed only tenders, never a product spec store.
- **P8** Every matchup ships `provenance="estimate"` + one card-level eid; per-spec provenance untracked.

### Partnerships
- **N1** Extraction captures 4 of 11 staging fields; `partner_kind`/`partner_country`/`ptype_raw`/
  `detected_lines` never populated.
- **N2** The relevance rule's first two branches are dead → **every** partnership is `kssl_relevance="context"`.
- **N3** Actively wrong: `partner_kind` always NULL → a Rafael/IAI licence renders "Domestic tie — lower
  sanctions risk" (`domain_pipeline.py:54-58`).
- **N4** Lossy identity + blanket overwrite: MoU→JV makes two cards; name variants split; a thin later
  report erases `deal_value`/`date`.
- **N5** No `write_evidence` (→ `/explain/partnership/*` 404s), no confidence, no corroboration.
- **N6** No partnership LLM task; `meaning` is a fixed template whose only branch is dead.

### Geography
- **G1** Last-writer-wins on `(competitor, country, raw name)`: variants split; a later "Offered" silently
  regresses "Contracted"; no first/last-seen anywhere.
- **G2** Binary Contracted/Offered only; no manufacturing/office/government dimension; `since_year`/`qty`/
  `note` never populated.
- **G3** Geo resolution has no text fallback (unlike partnerships one function up).
- **G4** Only aggregation is transient contested-countries; nothing aggregates one competitor's footprint;
  kg emits one `present_in` edge **per row**, never sets `first_seen`, wiped each rebuild.
- **G5** No geo evidence rows (`/explain` dead); `medium` guesses serve as `sourced`.

---

## 3. Gaps causing inconsistent outputs

- **Verdict nondeterminism:** substring keywords + keyword-order + identity-prior fallback → similar
  inputs diverge; LLM prose runs at temperature 0.2 with no seed, and the extraction fan-out is
  uncached (`extraction.py` `with_db(None)` path).
- **Label inside identity:** corroboration keys on `dir` and runs before classification → duplicates
  split, counts drift, any relabel re-splits groups.
- **Post-hoc justification:** enrich is handed the verdict and rationalises it; on validator failure it
  emits fixed boilerplate.
- **Missing features:** no event taxonomy, no product resolution, no KSSL market list — the facts needed
  to reason consistently are simply not gathered.

The fix is structural: make the verdict a **pure function of a canonical feature tuple**, gather the
missing features, and cache/seed everything (§9, §12).

---

## 4. Improved reasoning framework (overview)

Every intelligence area adopts the same shape:

```
   AGENT (reads)                 RULES (decide)              EVIDENCE (proves)
   ─────────────                 ──────────────              ─────────────────
   extract features   ───▶   deterministic gates /   ───▶   cited quote + eid
   + verbatim quotes         scores / rankings              per fired fact
                                    │
                             missing fact ⇒ "insufficient evidence", never a guess
```

- **Movement** → a feature checklist + a gate ladder (§5).
- **Positioning** → detect → resolve → category+size gate → compare → coverage-honest output (§6).
- **Partnerships** → capture the full feature set → importance-scoring rule → why-it-matters contract (§7).
- **Geography** → an evolving per-competitor×country footprint computed from existing rows (§8).
- Cross-cutting: confidence (§9), evidence/explainability (§10), anti-hallucination (§11).

---

## 5. Movement — Threat vs Watch decision tree

**Two labels only: Threat / Watch.** The label is a pure function of extracted facts. The AI never emits
or overrides it.

### 5.1 The feature checklist (gathered before any decision)

| # | Feature | Question | Values | Stored in (existing) |
|---|---------|----------|--------|----------------------|
| F1 | competitor | A tracked competitor? | ref id / unknown | `StgSignal.resolved_competitor_id` |
| F2 | event_type | What kind of event? | 14-value enum (5.2) | tag `et:<type>` |
| F3 | comp_products | Which competitor product(s)? | ref ids / none | `StgSignal.detected_products` (start reading it) |
| F4 | category_ids | What category? | RefCategory slugs | tag `cat:<slug>` |
| F5 | kssl_affected | KSSL products in that category? | ref ids / none | `SrvSignalDetail.facts` + tags |
| F6 | directness | How directly tied to KSSL? | matchup_direct > same_category > category_hint > none | tag `match:<level>` |
| F7 | country_status | A KSSL market? | active / pursuit / other / unknown | tag `mkt:<status>` |
| F8 | tender_link | An open tender KSSL fits? | tender id / none | tag `tender:<id>` |
| F9 | deal_value | A money figure? | numeric / none | `StgSignal.deal_value_num` |
| F10 | evidence_quotes | Verbatim proof per feature | text spans | `SrvEvidence.quote` |

**Directness (F6)** is the discriminating fact — all 8 categories are KSSL's own portfolio, so bare
category overlap is weak; the directness ladder carries the real signal:
`matchup_direct` (a curated head-to-head pair) ≫ `same_category` (resolved competitor product in a
category KSSL serves) ≫ `category_hint` (keyword only) ≫ `none`.

### 5.2 Event-type taxonomy (fact F2 — one per signal, closed set)

| event_type | Triggers | Doctrine mapping |
|---|---|---|
| `contract_award` | won / secures / selected / order placed | Threat-eligible (T1/T5) |
| `contract_loss` | lost / disqualified / cancelled / grounded | Watch + `opportunity` |
| `product_launch` | unveils / launches / first firing / rollout | Threat-eligible (T2) |
| `certification_trial` | user trials / qualification / DGQA clearance | Threat-eligible (T2, weaker) |
| `market_entry` | new-country subsidiary / plant / export clearance / office | Threat-eligible (T3) |
| `capacity_expansion` | new line / capex / factory expansion | Threat-eligible (T3, needs market overlap) |
| `partnership` | JV / MoU / licence / teaming | Threat-eligible (T4) |
| `acquisition` | acquires / stake purchase | Threat-eligible (T4) |
| `tender_activity` | RFP / RFI / EoI issued | Watch (market) unless competitor-linked (T5) |
| `rnd_tech` | patent / demo / R&D programme | Watch |
| `financial` | results / orderbook / funding | Watch |
| `org_people` | hiring / restructuring / leadership / CSR | Watch (explicit non-threat) |
| `incident_setback` | delay / failure / sanction / accident | Watch + `opportunity` |
| `other` | anything unmapped | Watch + `evidence_gap:event_type` |

The regex fallback gets a word-boundary keyword→event_type map so offline mode still emits the enum.

### 5.3 The gate ladder (evaluate top-down; first match wins; all matches recorded)

**Stop gates (force Watch — never guess):**
- **G0** competitor unresolved → **Watch + `evidence_gap:competitor`** (never a Threat without an actor).
- **G0b** event_type = `other` → **Watch + `evidence_gap:event_type`**.
- **G0c** event_type ∈ {contract_loss, incident_setback} → **Watch + `opportunity`** (former "fav").

**Threat gates (precedence T1 > T2 > T3 > T4 > T5):**

| Gate | Fires when ALL true | Meaning |
|------|---------------------|---------|
| **T1** | contract_award · directness ≥ same_category · country ∈ {active, pursuit} | Direct competitive loss/pressure in a KSSL market |
| **T2** | event ∈ {product_launch, certification_trial} · directness ≥ same_category | A competing product |
| **T3** | event ∈ {market_entry, capacity_expansion} · directness ≥ category_hint · country = active | Entering KSSL's market |
| **T4** | event ∈ {partnership, acquisition} · gained capability ∈ KSSL portfolio (F4 non-empty) · directness ≥ same_category | Strengthens direct competition |
| **T5** | event ∈ {contract_award, tender_activity} · tender_link ≠ none (KSSL fit ≥ medium) · competitor named on it | A contract KSSL could realistically pursue |

**Default → Watch + `monitor`.**

### 5.4 Edge rules & tie-breaks (where the old system fails)

| Situation | Rule |
|---|---|
| Multiple gates fire | Lowest-numbered gate labels; all fired gates recorded in tags (audit) |
| A gate conjunct's quote failed the substring check | Conjunct counts as **absent** — the gate does not fire (R2, §11) |
| contract_award, product unresolved, no tender link | **Watch + `evidence_gap:product`** (today: automatic Threat) |
| contract_award in a non-KSSL market | T1 fails on country → **Watch** — no direct impact |
| country_status = unknown | Treated as `other` for firing, but adds `evidence_gap:country` (confidence-capped) |
| Same corroboration group disagrees | Group-level re-evaluation wins (§9) |

### 5.5 Leaf contracts (what each outcome must contain)

Each leaf = `{label, rationale template, required evidence}`. Examples:

- **T1 → Threat:** *"{competitor} won {value} {category} business in {country}, a market KSSL
  {actively serves / is pursuing} with {kssl_products}. Direct competitive displacement."* —
  evidence: event quote + product-resolution eids + market-status eid.
- **T2 → Threat:** *"{competitor}'s {comp_product} competes head-to-head with KSSL's {kssl_product}
  ({matchup / shared category})."*
- **Watch + opportunity:** *"Setback for {competitor}: {event}. No direct KSSL impact; flags an opening
  for {kssl_products}."*
- **Watch + evidence_gap:** *"Insufficient evidence to assess KSSL impact: {missing feature(s)} could
  not be established from the source. Classified Watch pending corroboration."*
- **Watch + monitor:** *"Competitor-related, no direct KSSL impact: {why each nearest gate failed —
  the counter-factual, e.g. 'country Brazil is not a KSSL market'}."*

### 5.6 Fate of the retired "fav" label (signals only)

- Signals use `dir ∈ {threat, watch}`. Former-fav events → `watch` + `opportunity` tag; `rank_group`
  becomes "Watch — Opportunities" (string, no schema).
- Metrics strip: the "Favourable" tile becomes "Opportunities", counted by the tag (JSONB absorbs it).
  Opportunities rank above plain Watch within the group and get their own chip.
- `RefCompetitor.threat_level` is **removed as a classifier input** — display/priority metadata only.
- Untouched separate axes: `SrvMatchup.dir` ("who leads"), KG edge dir, anchor seed `threat_level`.

### 5.7 KSSL market list (the data gap behind F7) — **owner-authored**

KSSL's own geography is not stored today (`SrvGeoEntry` is competitor-only; `RefCountry` is a crawl-target
list including competitor home turf). Fix without schema: add to the anchor block in
`seed_data/watchlist_entities.json`:

```json
"anchor": { "id": "KSSL", "markets": { "active": ["india", "..."], "pursuit": ["saudi_arabia", "..."] } }
```

Loaded by `seed/loader.py` as KSSL geo rows (`stage` = Active/Pursuit), queryable for F7, renders on the
existing map. **The product owner authors and maintains the two country lists** (decision taken). A
dedicated `ref_kssl_markets` table would be cleaner but is a schema change — deferred.

---

## 6. Positioning — category-gated product comparison

**Stages: detect → resolve → category gate → size-class gate → equivalent selection → spec assembly →
compare → confidence + explain.** A comparison is *blocked, never forced* at any gate; every block state
is itself a published, explainable outcome.

### 6.1 Product detection & resolution — `resolve_product`
Mirror `resolve_competitor` exactly (exact id → longest-alias-first, word-boundary scan over **both**
product tables). Unresolved → explicit `{status:"unmapped", raw}`; never a fuzzy match. One resolver
consolidates the four places product strings appear today (`detected_products`, `StgGeo.product_name`,
vision labels, `RefMatchup.comp_name`). Unmapped strings accumulate into a **curation queue** (a query,
no new table); a product only enters the taxonomy via seed curation — never auto-inserted.

### 6.2 Category gate + size-class (the "small radar vs small radar" rule) — **spec-band, decided**
- Gate = candidates where `category_id == competitor product's category_id`. Cross-category is
  structurally impossible.
- **radar / EW are new seed categories** (add to `watchlist_products.json` + `_CATEGORY_NAMES` +
  `spec_slots.json`). No schema.
- **Size-class via spec-derived bands** (decided): `spec_slots.json` gains a `subclasses` section — per
  category an **anchor slot** + band table, e.g. `radar: anchor=detect_range_km, bands=[0–50 short,
  50–150 medium, 150+ long]`. Sub-class = the band of the product's anchor value. If **both** sides have
  the anchor → require the same band; if **either** lacks it → proceed flagged `subclass:"unverified"`
  with a confidence discount. Never silently compares across size.

**Gate outcome taxonomy (all published, explainable):** full comparison · size-class mismatch (blocked,
names both bands) · **capability whitespace** (competitor product, no KSSL counterpart — itself intel) ·
null-category curation gap · unmapped-product queue.

### 6.3 Spec acquisition — two-tier provenance
- **Tier A (curated, trusted):** generalise the seed file so `RefProductSpec` is populated for
  `product_side='competitor'` too (the column exists, unused). First tranche is nearly free — transcribe
  the `comp_num` values already inside the 8 curated matchups. Only curated rows live in `ref_*`.
- **Tier B (extracted, `estimate`):** PDF/page specs (already produced into `StgAssetAnalysis.extracted_specs`)
  become comparison-eligible **at assembly time** when their document resolves to a product; normalised via
  `spec_extract` slots, tagged `provenance='estimate'`, eid `att:`/`doc:`. **Never written into `ref_*`**
  (keeps the provenance split structural). Tier A always beats Tier B for the same slot.
- Slot vocabulary finally lands in `spec_slots.json` (extends the 3 built-ins) with per-slot **unit
  conversions** (fixes 18,000 kg vs 18 t) and tightened calibre keywords.

### 6.4 Comparison math — reuse the matchup engine
Product-vs-product is **symmetric better/worse by polarity** = the existing `_leader`/`_compute`
(`matchup_synthesis.py`), not the asymmetric tender `_score_product`. The dynamic path is a **pair
generator** feeding that engine; curated `ref_matchups` always override an auto-pair for the same pair.
`edge = clamp(50 + 12·(kssl_pts − comp_pts), 5..95)`, highlight ×2.

**Equivalent selection** (one competitor product, several in-category KSSL candidates):
`similarity = weighted mean of per-slot closeness (1 − |k−c|/max), anchor slot ×2`. Require `|shared
numeric slots| ≥ 2` AND `similarity ≥ 0.5` to auto-select the best; ties → more shared slots → curated
pair. Below threshold → "nearest candidate at 0.42 — below pairing threshold, flagged for curation", no
comparison published.

### 6.5 Missing-spec policy — kill the silent tie
- Fourth leader state `no_data` (string col + JSONB, no schema): 0 points, excluded from the denominator.
- Coverage denominator = category slot template ∪ observed slots — absences are *nameable*.
- **Mandatory coverage sentence** on every comparison: *"Compared 4 of 7 expected slots — missing: rate
  of fire, crew (competitor side); endurance (both sides)."*
- Zero coverage → no edge score; card publishes *"specs unavailable for X — no measured comparison
  possible"* with curated context lines only.

### 6.6 Auto-pair publication — **publish as estimate, decided**
Category + similarity gates passed → the comparison **publishes immediately** as `provenance='estimate'`
with the §6.7 confidence discount and coverage sentence. Positioning coverage tracks the full ~89-product
catalogue from day one; curated pairs remain first-class and override.

### 6.7 Comparison confidence + explainability
Decomposed (same `{factor,label,points,max}` shape signals use): coverage ≤40 (`40·compared/expected`) +
spec provenance ≤30 (A/A=30, mixed=18, B/B=10) + pair provenance ≤15 (curated=15, auto=8) + sub-class
verification ≤15. Stored as a `_summary` element appended to `edge_parts` JSONB (no schema). Per-spec
evidence eids (`spec:` for Tier A, `att:`/`doc:` for Tier B); add `"matchup"` to `_CONF_MODEL` so
`/explain` works; provenance stops being hard-coded `estimate` (all-Tier-A → `sourced`).

---

## 7. Partnerships — fill the fields that already exist

The staging table is already right; the pipeline around it is hollow. The fix is to **capture the
existing-but-empty columns** plus two data-driven seed files.

### 7.1 Feature set & acquisition

| Feature | Column (exists) | How obtained |
|---|---|---|
| Type | `rel_type` + `ptype_raw` | LLM emits `ptype_raw` verbatim; a deterministic keyword table canonicalises → `rel_type` (rule beats model) |
| Partner org | `partner_name` + `partner_id` | `resolve_partner`: (1) run competitor-resolution over the partner — **is-the-partner-also-a-competitor** check; (2) new seed `known_orgs.json` (~30 recurring OEMs); (3) normalised name, id NULL |
| Country | `partner_country` | known_orgs → extraction → NULL (stated, not guessed) |
| Partner kind | `partner_kind` | Foreign OEM / Domestic private / DPSU / Academia / Startup / Competitor-peer |
| Purpose | rule-derived | table over (rel_type × partner_kind × detected_lines): licence+OEM → tech_acquisition; supply+foreign → market_access; JV+domestic+mfg → capacity; MoU+academia → rd_pipeline |
| Technology / products | `detected_lines` | existing `_tech_domain`/`_category_hint` + `resolve_product` over the description |
| Description | `description` | the partnership sentence(s), not `doc.title` |

### 7.2 Type taxonomy + depth ladder (for merge)
`mou(1) < supply/investment(2) < license/rd_collab(3) < manufacturing(4) < jv(5) < acquisition(6)`.

### 7.3 Strategic-importance score (rule, decomposed)
```
importance = 20 (base: competitor + partner identified)
           + 25 if any detected_lines category ∈ KSSL portfolio
           + 10 if any tech-domain matches
           + 15 if partner_kind = Foreign OEM
           + 10 if partner resolves to a tracked competitor (alliance/consolidation)
           + 10 if rel_type ladder ≥ 5 (jv/acquisition); +7 if 3–4
           + 10 if deal value present
           clamp [5,95]
```
`kssl_relevance` derives from the score's **composition**: CORE if category-overlap fired; ADJACENT if
tech/OEM fired without it; context otherwise. The factor list is written as an evidence row so `/explain`
shows the arithmetic.

### 7.4 "Why it matters" contract (5 parts, each grounded)
Type (canonical + verbatim) · Purpose (rule enum) · **Threat** (real `detected_lines`, not "non-core") ·
**Opening** (if the partner backs ≥2 tracked rivals, cite the existing `agg:shared_partner` chokepoint) ·
**Dependency** (Foreign-OEM IP/sanctions vs domestic vs competitor-peer — correct now that `partner_kind`
is real). NULL features render as "partner country unreported", never fabricated.

### 7.5 Merge policy (replaces last-writer-wins)
New key `(competitor_id, partner_canonical)`; `rel_type` leaves the key and evolves. Field policies:
`keep_earliest`(date) · `non_null_latest`(value/country/kind — null never erases) · `ladder_max`(rel_type;
downgrade only on termination keywords) · `set_union`(detected_lines) · `recompute`(relevance/meaning/
importance). `stg_partnerships` stays the append-only timeline.

### 7.6 Trust spine
`write_evidence(target_kind="partnership")` (makes `/explain` work — today 404). Corroboration mirrored
with key `(competitor_id, partner_canonical)`. `confidence.score()` reused. **Confidence storage** is the
one schema gap — interim in the evidence quote; real columns are the deferred decision (§12).

---

## 8. Geography — the evolving footprint

### 8.1 Presence-type taxonomy + deterministic triggers

| presence_type | Trigger | Source |
|---|---|---|
| `operates` | any accepted row (the floor) | all |
| `contract` | stage=Contracted OR value present OR kw order/contract/win/delivered | geo |
| `pursuing` | stage=Offered OR kw offered/bid/trials/evaluation | geo |
| `manufacturing` | kw plant/facility/production line/assembly/MRO | geo.note, events, text |
| `partnership` | `SrvPartnership.country == country` | partnerships |
| `government` | kw MoD/ministry/G2G/LoA/offset/cooperation agreement | signals |
| `recently_expanded` | first_seen within 180 days (derived) | computed |

Stage ladder `Offered < Contracted < Manufacturing`, monotonic — regresses only on explicit loss
keywords (fixes silent regression). `ExtractGeo.stage` widens; `since_year`/`qty`/`note` start being
filled. Geo key uses the **canonical resolved product name** so "Caesar"/"CAESAR 6x6" stop splitting.

### 8.2 The footprint — a computed view, no new tables
Staging is the append-only event log this feature needs; `SrvGeoEntry` is only the current-state cache.
The rollup per (competitor × country) is assembled from `SrvGeoEntry` + `stg_geo ⨝ stg_documents`
(timeline) + `SrvPartnership` + `stg_signals`:

```
{ country, presence:[...], stage_now, products:[canonical], first_seen, last_seen,
  n_reports, n_sources, contract_values, partnerships, recently_expanded, contested,
  confidence:{score,band,parts}, evidence:[geo:, part:, sig:, ...] }
```

Cell confidence reuses `confidence.score()` (tier = best member, independent_sources = distinct
source_ids = geo corroboration, freshness = last_seen). **Served at request time**:
`GET /geo/footprint?competitor=` next to `/geo` (small volume; the codebase already recomputes the whole
graph each batch). A materialised `srv_geo_footprint` table is the deferred alternative (mandatory only
if `stg_*` is ever pruned — the design assumes append-only staging).

kg improvement (no schema): collapse `present_in` to **one edge per (competitor, country)** with
`{stages, products, n_rows}` and pass `first_seen` (the column/param already exist, never used for geo).

### 8.3 First/last-seen — what's possible without schema
Possible: `first_seen = min(doc.published_at)`, `last_seen = max`, `recently_expanded`, stage timeline —
all from append-only staging joined to documents. **Impossible without schema (stated):** `SrvGeoEntry`
itself can't carry first/last-seen (no timestamp columns) — only the computed view has time depth; and if
`stg_*` is ever pruned, footprint history dies with it.

### 8.4 Evidence
`write_evidence(target_kind="geo_entry")`; footprint cells return member eids
(`agg:footprint:<comp>:<country>`); provenance tightened `high→sourced`, `medium|low→estimate` (stops
laundering guesses as sourced).

---

## 9. Confidence scoring strategy

Keep the 4-part formula and clamp. Four changes:

| # | Change | Rule |
|---|---|---|
| C1 | Stop hard-coding provenance | Part 4 becomes **grounding (0–15)**: 15 = doc present AND every fired-gate conjunct has a surviving quote; 8 = doc present + ≥1 `evidence_gap`; 5 = no doc / competitor unresolved |
| C2 | Certainty caps the band | Any `evidence_gap:*` ⇒ total capped at 59 — a degraded classification can never read "high" |
| C3 | Confidence enters rank | `recompute_ranks` key → (dir_weight, band_bucket high2/med1/low0, recency) — using band buckets so corroboration drift doesn't churn the feed |
| C4 | Fix the corroboration key | `competitor|country|dir|value` → `competitor|country|event_type|value` — removes the label from identity and fixes the runs-before-classification undercount (event_type exists pre-classification) |

**Decided:** a well-evidenced single-source Threat stays a **Threat with an `unconfirmed` tag** and a low
band — impact and likelihood are separate axes. The `evidence_gap` path (C2 + G0) already forces Watch
when the *features* are unsupported.

---

## 10. Evidence & explainability framework

- **Uniform `/explain`:** extend `write_evidence` to `partnership`, `geo_entry`, and per-spec `matchup`
  targets; add those kinds to `_CONF_MODEL`. Today only signal/matchup/synthesis/field_pattern/kg_edge/
  document_asset write evidence, and `/explain/partnership|geo/*` 404.
- **Rationale storage without schema:** namespaced tags (`gate:`, `et:`, `match:`, `mkt:`, `evidence_gap:`)
  + `facts` JSONB rows (Classification / Why / KSSL products affected / Competitor products / Evidence /
  Why-not / Gaps) + `why_text` whose deterministic fallback is the **rendered leaf template**, not
  generic boilerplate.
- **Audit-grade minimum** — a classification is invalid without: label + gate id + ruleset version; the
  full feature tuple; the why (mechanism of KSSL impact); the why-not counter-factual (Watch); affected
  KSSL + competitor products (or an explicit "none resolved"); ≥1 verbatim quote per fired conjunct;
  every gap declared as a facts row.

---

## 11. Anti-hallucination / never-guess rules

Extends the existing pattern (numbers-grounded, cites-required, drop-uncited, closed competitor ids) into
classification:

| # | Rule |
|---|---|
| R1 | The label is never LLM-emitted or prose-overridable; the gate evaluator replaces the keyword stub |
| R2 | **Feature-must-cite:** a gate conjunct counts only if its quote survives a literal substring check against the source; failed quote ⇒ feature absent ⇒ gate cannot fire |
| R3 | **Closed-world resolution:** competitor and product ids accepted only if they resolve against ref aliases (word-boundary); unresolved mentions are recorded but never feed a gate |
| R4 | **Threat ⇔ gate:** `dir=threat` is valid only if ≥1 gate fired AND each conjunct has an evidence row; publish-time assert, else demote to Watch + `evidence_gap:gate_evidence` + log |
| R5 | **Closed enums everywhere:** out-of-taxonomy event_type → `other` (can never fire a threat) |
| R6 | **Insufficient evidence is a first-class output:** every missing critical feature emits an `evidence_gap` tag + facts row + the C2 cap — no silent default |
| R7 | **Fix prose collapse:** widen enrich's evidence text to include a capped `main_text` excerpt (the text quotes were validated against); on failure fall back to the leaf template, not generic boilerplate |
| R8 | **Prose cannot upgrade the verdict:** all verdict-bearing fields are rule-written after enrichment |
| R9 | **Uniform explainability:** every published signal writes a `field="dir"` evidence row — `/explain` is never empty |
| R10 | **Fail closed on empty ref data:** if portfolio/matchup/market tables are empty, gates cannot fire ⇒ everything is Watch + `evidence_gap:portfolio` |

### Agentic layer (approved scope) — "agents read, rules decide"
Agents live only on the perception side; **every verdict, score, rank, and number stays deterministic.**
- **B · Adversarial evidence-verifier** (hot path, +1 call/signal): a skeptic agent tries to *refute*
  each extracted feature against the source. A semantic layer above R2 — a feature survives only if not
  refuted. This is §11 upgraded from a rule to rule+agent.
- **C · Agentic competitor synthesis** (cold, per-competitor): research → draft → self-critique →
  drop-uncited → finalise; upgrades today's single-shot synthesis.
- **D · PDF/brochure spec-miner** (cold, per-asset): multi-step read to fill Tier-B competitor specs (§6).
- **E · Curation-queue proposer** (cold, human-in-loop): proposes canonical name/category/aliases for
  unmapped products/partners; **agent proposes, human/rule commits** (never auto-insert).
- **Rejected:** a per-area specialist extraction council on the hot path (cost + determinism) — extraction
  stays a single structured call. **Never agentic:** the gates, confidence, edge-score, rank, any number.

---

## 12. Determinism & consistency recommendations + roadmap

### 12.1 Why similar inputs diverge today, and the fix
| Cause | Fix |
|---|---|
| Substring keywords, keyword-order, identity-prior fallback | Pure-function `label = f(feature tuple)`; word-boundary matching everywhere |
| LLM sampling (temp 0.2, no seed, uncached fan-out) | temperature 0 + fixed seed for feature extraction; route the fan-out through the cache |
| Label inside the corroboration key, computed pre-classification | C4 (key on event_type); evaluate gates **once per corroboration group** and give every member the group verdict |
| Post-hoc prose | Verdict fixed before enrichment; R7/R8 |
| Silent drift between runs | Feature-tuple hash + **ruleset version** (`gate:T2@v1`); reprocessing is an explicit, logged migration |
| No regression safety | Golden **feature-tuple → label** tests (pure-function, no LLM) + re-goldened end-to-end cases |

### 12.2 Phased roadmap (build order; each phase independent)

| Phase | Scope | Agent | Acceptance criteria |
|---|---|---|---|
| **P1 — Movement** | feature checklist, event taxonomy, gate ladder, fav→opportunity, KSSL market seed, confidence C1–C4, R1–R10 | verifier (B) | Every signal shows label + gate + why + evidence; the 6 example events (§12.3) land deterministically; golden feature-tuple tests pass; no `fav` in signal output |
| **P2 — Positioning** | `resolve_product`, radar/EW seed, spec-band sub-class, Tier-A/B specs, dynamic pairing, coverage policy | spec-miner (D) + curation (E) | Auto-pairs publish as estimate with coverage sentence; no silent ties; cross-category/size comparison impossible; `/explain` works for matchups |
| **P3 — Partnerships** | full feature capture, `resolve_partner`, importance score, merge policy, evidence | curation (E) | No partnership renders the dead template; `/explain/partnership` works; MoU→JV merges to one card; importance arithmetic visible |
| **P4 — Geography** | presence taxonomy, footprint view, kg collapse, evidence, provenance | — | `GET /geo/footprint` returns an evolving per-competitor view with first/last-seen and cited presence badges |
| **Synthesis** | agentic dossier | synthesis (C) | Drop-uncited enforced; every claim cites evidence in the pack |

**Build-time (decided):** one implementation agent per phase in parallel (independent domains), coordinated
and merged centrally.

### 12.3 Decision-tree dry-run (proof of determinism — to expand during P1)
| Event | Features | Outcome |
|---|---|---|
| "Competitor wins ₹500cr 155mm order in India" | contract_award · same_category · country=active | **Threat (T1)** |
| "Competitor unveils new 155mm gun" | product_launch · same_category | **Threat (T2)** |
| "Competitor opens artillery plant in India" | market_entry · category_hint · country=active | **Threat (T3)** |
| "Competitor restructures its board" | org_people | **Watch (monitor)** |
| "Competitor runs a CSR tree-planting drive" | org_people | **Watch (monitor)** |
| "Competitor wins a naval contract in Argentina" | contract_award · directness=none/other market | **Watch** (T1 fails on directness+country) |

### 12.4 Deferred decisions (schema-blocked — flagged, not built)
- Confidence columns on `SrvPartnership` / `SrvGeoEntry` / `SrvMatchup` (interim: evidence quote +
  `edge_parts` summary). The single genuine schema change if adopted.
- First-class `event_type` / `gate_id` columns (namespaced tags absorb them for now).
- `ref_kssl_markets` table (seed `anchor.markets` covers it for now).
- Materialised `srv_geo_footprint` table (only needed if staging is ever pruned).

---

*Competitive Player redesign · deterministic-first, agent-assisted · numbers & verdicts never
model-computed · every conclusion traceable via `/explain` · insufficient evidence is a first-class
output, never a guess.*
