# Mallory — How the Intelligence Engine Works

> **Written 2026-07-02** | Based on real data processed from the crawler through L2 into serving tables.
> Every example in this document uses actual records crawled today from live defense news.

---

## Table of Contents

1. [Overview — What this engine actually does](#1-overview--what-this-engine-actually-does)
2. [The three layers of data](#2-the-three-layers-of-data)
3. [Database — All 27 tables](#3-database--all-27-tables)
4. [Data flow — Crawler to Intelligence](#4-data-flow--crawler-to-intelligence)
5. [Service catalog — Every stage explained with real data](#5-service-catalog)
   - [S-05: Entity Resolution](#s-05-entity-resolution)
   - [S-07: Signal Classification](#s-07-signal-classification)
   - [S-09: Signal Enrichment](#s-09-signal-enrichment)
   - [S-10: Rank Computation](#s-10-rank-computation)
   - [S-11: Overview Metrics](#s-11-overview-metrics)
   - [S-12: Tender Spec Parsing](#s-12-tender-spec-parsing)
   - [S-13: Tender Scoring & Verdict](#s-13-tender-scoring--verdict)
   - [Partnership Processing](#partnership-processing)
   - [Geo Footprint Processing](#geo-footprint-processing)
   - [Innovation Processing](#innovation-processing)
   - [S-25: Mallory Chat (RAG)](#s-25-mallory-chat-rag)
   - [S-26: CEO Report](#s-26-ceo-report)
   - [Auto-Processor (5 min scheduler)](#auto-processor)
6. [Critical crawler fields — What feeds the engine](#6-critical-crawler-fields--what-feeds-the-engine)
7. [End-to-end trace — One real signal from crawler to frontend](#7-end-to-end-trace--one-real-signal-from-crawler-to-frontend)

---

## 1. Overview — What this engine actually does

The Mallory Layer 2 engine takes **raw web-crawled news and tenders** about defense companies and turns them into **ranked, scored, analyzed competitive intelligence**. Everything is framed "vs KSSL" (Kalyani Strategic Systems Limited), the anchor company.

**Input:** Crawler POSTs `{document, record}` — article text + structured fields (competitor name, event summary, deal value, country, etc.)

**Output:** Pre-computed serving tables the frontend reads with zero math — signal cards with threat levels and "so what" analysis, tenders scored against KSSL products, partnerships tagged by relevance, innovation tracked vs KSSL's roadmap.

---

## 2. The three layers of data

```
┌─────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  stg_* (staging) │ ──► │  Pipeline        │ ──► │  srv_* (serving)  │
│  Raw crawler      │     │  Compute         │     │  Pre-computed     │
│  input            │     │  + classify      │     │  for frontend     │
│  proc_status:     │     │  + score         │     │  (read-only)      │
│  received         │     │  + enrich        │     │                   │
└─────────────────┘     │  + rank          │     └──────────────────┘
                        │  + measure       │
                        └──────────────────┘
                              ▲
                              │
                  ┌──────────────────┐
                  │  ref_* (reference)│
                  │  Static seed:     │
                  │  competitors,     │
                  │  products, specs, │
                  │  categories,      │
                  │  countries        │
                  └──────────────────┘
```

- **`ref_*`** — Static baseline loaded from `seed_data/*.json`. 34 competitors, 27 KSSL products, ~80 competitor products, product specs, categories, tech domains, countries. Never changes during runtime.

- **`stg_*`** — Append-only staging tables. Every crawler record lands here with `proc_status='received'`. Contains exactly what the crawler sent plus L2-computed columns.

- **`srv_*`** — Read-only serving tables. Every value the frontend needs is a literal column here — direction, rank, "so what" text, fit percentages, relevance tags. The client never computes anything.

---

## 3. Database — All 27 tables

There are **27 tables** in three tiers. Here's every one, what it stores, and today's live data count.

### REFERENCE (ref_) — Static "vs KSSL" baseline — 183 rows across 7 tables

Seed data loaded once at startup from `seed_data/*.json`. Never changes during runtime.

| # | Table | Rows | Purpose |
|---|---|---|---|
| 1 | `ref_categories` | 8 | Product categories: artillery, armoured, small_arms, ammunition, missiles_ad, naval, uav, materials |
| 2 | `ref_countries` | 25 | Target procurement countries: India, USA, Armenia, Nigeria, UAE, France, Germany, UK, etc. |
| 3 | `ref_tech_domains` | 8 | Tech domains with keyword maps: artillery, armoured, small_arms, ammunition, missiles_ad, naval, uav, materials |
| 4 | `ref_competitors` | 34 | Tracked competitors: KSSL as anchor + 31 rivals + 2 partner nodes. Each has id, name, aliases, hq_country, threat_level, priority (P1/P2/P3) |
| 5 | `ref_kssl_products` | 27 | KSSL's own products: ATAGS, MArG 155, M4, CQB Carbine, Spike KRAS, Nagastra-1/2/3, Bharat-52, etc. Keyed by category |
| 6 | `ref_competitor_products` | 89 | Competitor products: CAESAR 6x6, K9 Vajra, ATMOS 2000, etc. Keyed by competitor + category |
| 7 | `ref_product_specs` | 15 | Per-product numeric specs (calibre, range, weight) with polarity (higher_better/lower_better). Used by tender scoring engine |

### STAGING (stg_) — Crawler input, append-only — 212 rows across 7 tables

Written by `POST /ingest/v1/{record_type}`. Each row starts with `proc_status='received'` and advances through the pipeline. Columns marked "L2-computed" are filled by the pipeline, not the crawler.

| # | Table | Rows | Purpose |
|---|---|---|---|
| 8 | `stg_documents` | 85 | Source documents: url, content_hash, title, main_text, images[storage_path], screenshot, attachments, entities_detected. The parent of all other staging rows (FK: document_id). Dedup key: id (sha1 of url) |
| 9 | `stg_signals` | 69 | Competitive signals: stream, competitor_id, event_summary, detected_country, tech_domain, deal_value. **L2-computed:** resolved_competitor_id, dir, lens, tags, dedup_group, proc_status |
| 10 | `stg_tenders` | 2 | Procurement tenders: title, issuer, country, category_hint, value_raw/num/currency, deadline_date, requirement_text, requirement_fields[]. **L2-computed:** value_usd, category_id, proc_status |
| 11 | `stg_partnerships` | 2 | Competitor alliances: competitor_id, partner_name, partner_country, partner_kind, rel_type, description, detected_lines[]. **L2-computed:** kssl_relevance, proc_status |
| 12 | `stg_geo` | 1 | Geographic footprints: competitor_id, country, product_name, product_category, stage, confidence. **L2-computed:** proc_status |
| 13 | `stg_innovation` | 12 | Technology developments: tech_domain, title, competitor_id, driver, maturity_hint, horizon_hint, description. **L2-computed:** proc_status |
| 14 | `stg_company_events` | 41 | Company events: competitor_id, event_type, headline, deal_value, date_of_event. ⚠️ NOT processed by pipeline — sits in staging forever unless processing logic is added |

### SERVING (srv_) — Read-only output for frontend — 137 rows across 13 tables

Pre-computed, denormalized. Every value the UI needs is a literal column. The frontend never computes a score, rank, or color.

| # | Table | Rows | Purpose |
|---|---|---|---|
| 15 | `srv_signals` | 69 | One row = one overview feed card. Pre-computed: pillar, dir (threat/watch/fav), rank (sorted), rank_group, title, meta (domain·company·value), company, lens, sowhat (1-line analysis), tags, ago_display, source_url |
| 16 | `srv_signal_details` | 69 | 1:1 with srv_signals. Right-panel detail: facts[][], what_text, why_text, lens_reads[][], actions[][], suggest[] (follow-up questions), source_url |
| 17 | `srv_tenders` | 2 | Normalized tenders: title, issuer, country, category, value_display, value_usd (converted), qty, deadline_date, dl_days (computed daily), req_note, requirements[][], lean (go/maybe/pass), lean_text, status (open/closing/closed) |
| 18 | `srv_tender_matches` | 8 | Per-tender, per-KSSL-product scoring: tender_id, kssl_product_id, kssl_product_name, fit_pct, fit_level (high/medium/low), match_lines[][up/down + text] |
| 19 | `srv_overview_metrics` | 3 | One row per pillar: { competitive, market, technology }. Each has metrics[]: Threats/Watch/Favourable/Total counts with colors and filters |
| 20 | `srv_matchups` | 8 | Head-to-head product benchmarks: category, dir (who leads), comp_name vs kssl_name, edge_score (0-100), adv_comp[], adv_kssl[], verdict. Loaded from seed_data/matchups.json |
| 21 | `srv_matchup_specs` | 27 | Per-spec comparison rows: matchup_id, spec_label, comp_value, kssl_value, leader (comp/kssl/tie). Loaded from seed |
| 22 | `srv_geo_entries` | 1 | Geographic deployment data: competitor_id, competitor_name, country, product_name, category, contract_value, since_year, qty, stage, note, provenance (sourced/estimate) |
| 23 | `srv_partnerships` | 2 | Processed partnerships: competitor_id/name, partner_name/kind, rel_type, deal_value, kssl_relevance (CORE/ADJACENT/context), meaning (Threat/Opening/Dependency analysis), provenance |
| 24 | `srv_innovation` | 12 | Innovation cards: tech_domain_id, title, maturity, gap_vs_kssl (ahead/parity/behind), driver, horizon, body, impact, action, provenance |
| 25 | `srv_patents` | 6 | Patent records: competitor_id, tech_domain_id, jurisdiction, title, status, filed_date, assignee, abstract, kssl_relevance. Loaded from seed |
| 26 | `srv_competitor_synthesis` | 6 | Per-competitor strategic synthesis: thesis, strat_pattern, strat_sowhat, vulnerabilities[{title, intel}], predictions[], moves[]. Loaded from seed |
| 27 | `srv_field_patterns` | 5 | Cross-cutting competitive-pattern observations: title, summary, exceptions. Loaded from seed |

### Data flow summary across tables

```
seed_data/*.json ──────────────────────────► ref_* (7 tables, 183 rows, loaded once)
                                               │
Crawler POSTs ──► stg_documents (85)            │  S-05 entity resolution reads ref_competitors
                  ├─ stg_signals (69) ──────────┤  S-07/S-09/S-10 use ref_competitor + threat_level
                  ├─ stg_tenders (2) ───────────┤  S-12/S-13 use ref_kssl_products + ref_product_specs
                  ├─ stg_partnerships (2) ──────┤
                  ├─ stg_geo (1) ───────────────┤  Pipeline runner processes received → published
                  ├─ stg_innovation (12) ───────┤
                  └─ stg_company_events (41)    │  ⚠️ NOT processed (no pipeline logic)
                                               │
                                               ▼
                  srv_signals (69) + srv_signal_details (69)
                  srv_tenders (2) + srv_tender_matches (8)
                  srv_partnerships (2)
                  srv_geo_entries (1)
                  srv_innovation (12)
                  srv_overview_metrics (3)  ← recomputed after every pipeline run
```

---

## 4. Data flow — Crawler to Intelligence

```
CRAWLER (Layer 1)                     L2 (this engine)                    FRONTEND (Layer 3)
─────────────────                     ────────────────                    ──────────────────

Crawls defense news                POST /ingest/v1/page
Extracts entities + text  ──────►  stg_documents + stg_signals
Saves screenshots/images            stg_tenders + stg_partnerships
                                      stg_geo + stg_innovation
                                      stg_company_events
                                              │
                                    [Auto-pipeline runs every 5 min]
                                              │
                                              ▼
                                    S-05: Entity resolution
                                    S-07: Classify (threat/watch/fav)
                                    S-09: Enrich (so what, actions)
                                    S-10: Rank (within pillar)
                                    S-11: Metrics (counts per pillar)
                                    S-12: Tender spec parsing
                                    S-13: Tender scoring + verdict
                                    Partnership/Geo/Innovation processing
                                              │
                                              ▼
                                    srv_signals + srv_signal_details
                                    srv_tenders + srv_tender_matches    GET /api/v1/signals
                                    srv_partnerships              ◄───  GET /api/v1/tenders
                                    srv_geo_entries                     GET /api/v1/partnerships
                                    srv_innovation                     GET /api/v1/competitors
                                    srv_overview_metrics               (renders cards, zero math)
                                              │
                                    POST /api/v1/mallory/chat
                                    (RAG over serving data)
                                    POST /api/v1/reports/ceo
```

---

## 5. Service catalog

### S-05: Entity Resolution

**File:** `services/entity_resolution.py`

**What it does:** Confirms which competitor a signal is about. The crawler sends a `competitor_id` like `"KNDS"` — this service verifies it exists in our watchlist or finds it by scanning the article text.

**Real example from today:**

```
Crawler sent: competitor_id="KNDS"
               document.main_text="KNDS remporte une commande de CAESAR 6x6 pour le Nigeria..."

S-05: "KNDS" → exact match in ref_competitors → confirmed
      (If the crawler had sent competitor_id=null, S-05 would scan main_text
       and find "KNDS" / "Nexter" from the competitor aliases list.)
```

**Code flow:**
1. Check if `competitor_id` exists in `ref_competitors` — 1 SQL query
2. If yes → return it. Done.
3. If no → build alias index from `ref_competitors` (name + all aliases, sorted longest-first so "Larsen & Toubro" matches before "L&T")
4. Scan `main_text.lower()` for each alias substring
5. Return matched ID or None

**Why it matters:** Every downstream service depends on knowing WHO this signal is about. Wrong entity = wrong classification = wrong intelligence.

---

### S-07: Signal Classification

**File:** `services/signal_pipeline.py` → calls `llm.classify_signal()`

**What it does:** Reads the event summary and tags every signal as **threat**, **watch**, or **favourable** vs KSSL. Also assigns a **lens** (which angle to view it from) and **tags**.

**Real example from today:**

| Crawler sent | Classification | Why |
|---|---|---|
| `"KNDS wins CAESAR artillery contract in Nigeria"` | **threat** | "wins" → keyword match. Competitor gains market share on artillery — KSSL's ATAGS competes here. |
| `"Armenia issues tender for 155mm artillery systems"` | **watch** | Market stream + "tender" → an opening, not yet won by anyone. |
| `"Bharat-52 specifications (KSSL product page)"` | **favourable** | Article about KSSL's own product → favorable signal |
| `"India raises defence budget capital outlay"` | **watch** | Market stream, no winner/loser yet, but creates opportunity |

**Code flow (stub LLM):**
1. Check `event_summary` for win keywords: "won", "secured", "awarded", "acquires" → **threat**
2. Check for loss keywords: "delay", "fails", "lost", "setback" → **favourable**
3. Market stream + tender words ("RFP", "seeks", "budget", "closing") → **watch**
4. Technology stream → default to competitor's `threat_level` from seed
5. Assign lens: competitive→BENCHMARK, market→MARKET/DEMAND, technology→TECH MIGRATION
6. Add tags: direction + "opening" (if tender-related), "atstake" (if competitive), "deadline" (if closing)

**Code flow (OpenRouter LLM, when live):**
1. Sends system prompt: *"You are KSSL's competitive analyst. Classify this {stream} signal vs KSSL. Return JSON: {dir, lens, tags}"*
2. LLM reads event_summary with full context understanding — not just keyword matching
3. Falls back to stub on API error

**Today's classification results across 69 signals:**

| Direction | Count | Meaning |
|---|---|---|
| threat | 27 | Competitor wins, deals, or advances that pressure KSSL |
| watch | 41 | Openings, tenders, market movements, general competitor activity |
| fav | 1 | Article about KSSL's own product (Bharat-52) |

---

### S-09: Signal Enrichment

**File:** `services/signal_pipeline.py` → calls `llm.enrich_signal()`

**What it does:** This is where the real intelligence is generated. Takes the classified signal and writes the "so what" — why this matters to KSSL, what KSSL should do about it, and what to investigate next.

**Real example from today — Signal #21 (KNDS wins Nigeria CAESAR order):**

```
INPUT from crawler:
  event_summary: "KNDS wins CAESAR 6x6 order for Nigeria — 18 units, 155mm"
  stream: "competitive"
  company: "KNDS (Nexter / KMW)" (resolved by S-05)
  dir: "threat" (classified by S-07)
  facts: [Company: KNDS, Domain: artillery, Country: Nigeria]

OUTPUT (written to srv_signal_details):

  sowhat (1-liner): "This strengthens KNDS (Nexter / KMW) on a line KSSL
                     contests — expect direct pressure on KSSL's bids and pricing."

  what_text: Full event summary from crawler

  why_text: "Read against KSSL's portfolio, this moves the competitive balance
             in competitive. This strengthens KNDS on a line KSSL contests —
             expect direct pressure on KSSL's bids and pricing."

  lens_reads (multi-angle analysis):
    [BENCHMARK] "This strengthens KNDS on a line KSSL contests — expect
                 direct pressure on KSSL's bids and pricing."
    [POLICY/OFFSET] "Indigenous-content and offset rules remain KSSL's
                     structural lever here."

  actions (what KSSL should do):
    [Counter] "Brief KSSL BD on a positioning response within the week."
    [Benchmark] "Compare the named capability against KSSL's equivalent product."

  suggest (follow-up questions for Mallory chat):
    "What does this mean for KSSL?"
    "Who else is affected?"
    "Show the head-to-head."
```

**Code flow:**
1. Builds `facts` array from crawler fields: Company, Domain, Country, Value
2. Calls `llm.enrich_signal()` with all inputs
3. Stub path: direction-specific template:
   - threat → *"This strengthens X on a line KSSL contests..."*
   - watch → *"Not an immediate hit to KSSL, but it shifts the field..."*
   - fav → *"A stumble for X opens room for KSSL to press its indigenous-IP..."*
4. Writes `srv_signal_details` row — the right-panel detail the frontend shows when you click a signal card

---

### S-10: Rank Computation

**File:** `services/signal_pipeline.py` → `recompute_ranks()`

**What it does:** Sorts all signals within each pillar so the most urgent things appear first. Rerun after every processing batch.

**Real example from today — Competitive pillar top 5:**

| Rank | Company | Title | Direction |
|---|---|---|---|
| 1 | Adani Defence | Indian stocks defy logic... | threat |
| 2 | Larsen & Toubro | Air Marshal takes Southern Air Command | threat |
| 3 | Larsen & Toubro | ET Edge SCM Fest 2026 concludes | threat |
| 4 | Adani Defence | US judge orders DOJ justification | threat |
| 5 | KNDS | KNDS wins CAESAR order for Nigeria | threat |

**Code flow:**
1. `SELECT * FROM srv_signals` — group by pillar
2. Sort within each pillar:
   - Primary: direction weight (threat=3, watch=2, fav=1) — highest first
   - Secondary: `published_at` — newest first
3. Assign sequential rank starting from 1
4. Rank group label: threats→"Priority — Threats", watch→"Watch", fav→"Favourable"

**Note:** The rank ordering shows some noise (generic stock market articles ranked above specific defense intel) because the stub LLM classifies everything with keyword-matching and the `published_at` dates are close together. With OpenRouter enabled, classification quality and ranking precision improve significantly.

---

### S-11: Overview Metrics

**File:** `services/metrics.py`

**What it does:** Pre-computes the metric strip that appears at the top of each pillar view in the frontend.

**Real example from today:**

```
SELECT pillar, dir, COUNT(*) FROM srv_signals GROUP BY pillar, dir

Result:
  competitive: threats=27, watch=41, fav=1  → Total=69
  market: threats=3, watch=3                 → Total=6
  technology: threats=2, watch=4, fav=1      → Total=7

Written to srv_overview_metrics:
  { pillar: "competitive",
    metrics: [
      { label: "Threats", value: 27, color: "var(--threat)", filter: "threat" },
      { label: "Watch", value: 41, color: "var(--watch)", filter: "watch" },
      { label: "Favourable", value: 1, color: "var(--fav)", filter: "fav" },
      { label: "Total signals", value: 69, color: "var(--ink)", filter: "all" }
    ]
  }
```

**Frontend renders:** `Threats: 27  |  Watch: 41  |  Favourable: 1  |  Total: 69`

---

### S-12: Tender Spec Parsing

**File:** `services/tender_scoring.py`

**What it does:** Reads the crawler's `requirement_fields` and extracts normalized spec values to compare against KSSL products.

**Real example from today — Tender #1 (MoD India 155mm Mounted Gun System):**

```
Crawler sent:
  category_hint: "artillery"
  requirement_fields: []   ← CRITICAL ISSUE: empty!
```

**This is a problem.** The crawler did NOT send requirement_fields with spec values. Without structured requirement fields, the scoring engine can't extract numbers and all matches default to the base score.

**What the crawler SHOULD have sent:**
```json
"requirement_fields": [
  {"label": "Calibre", "value": "155"},
  {"label": "Range", "value": "≥ 40 km"},
  {"label": "Weight", "value": "≤ 18 t"}
]
```

**With proper fields, the code would:**

1. **`_slot_for(label)`** — maps labels to normalized slots:
   - `"Calibre"`, `"System"`, `"Gun"` → `calibre_mm`
   - `"Range"` → `range_km`
   - `"Weight"`, `"Mass"` → `weight_t`

2. **`_first_number(value)`** — extracts numbers: `"≥ 40 km"` → `40.0`, `"≤ 18 t"` → `18.0`, `"155"` → `155.0`

3. **`_required_op(value)`** — detects the operator: `">= 40 km"` → `>=`, `"≤ 18 t"` → `<=`, bare number → `==`

4. **`_kssl_specs(db, product_id)`** — loads KSSL product specs from `ref_product_specs`

5. **`_score_product(reqs, specs)`** — the core scoring algorithm

**Because requirement_fields was empty today, ALL matches scored exactly 55% (base score only):**

| KSSL Product | Fit % | Fit Level |
|---|---|---|
| ATAGS / Bharat 52 | 55% | medium |
| MArG 155 | 55% | medium |
| Bharat 45 | 55% | medium |
| Bharat ULH | 55% | medium |
| Garuda 105 | 55% | medium |

**With proper requirement_fields, this would look like:**
```
Requirement: Range ≥ 40km
  ATAGS range = 48.1km → MEETS → +14 points → 69%
  MArG 155 range = 30km → FAILS → -8 points → 47%

Requirement: Weight ≤ 18t
  ATAGS weight = 18t → MEETS → +14 → 83% → HIGH FIT
  MArG 155 weight = 18t → MEETS → +14 → 61% → MEDIUM FIT
```

**This is the single most important fix the crawler team needs to make.**

---

### S-13: Tender Scoring & Verdict

**File:** `services/tender_scoring.py`

**What it does:** Runs the spec scoring engine (S-12) on every KSSL product in the tender's category, decides go/maybe/pass, and computes timeline urgency.

**Real example from today:**

```
Tender #1: MoD India — 155mm 52-cal Mounted Gun System
  category: artillery
  value: Rs 6,500 cr → $783,132,530 USD (INR ÷ 83)
  deadline: 2026-07-08 → dl_days: 6 → status: "closing"
  lean: "maybe" (all matches at 55%)

Tender #2: SAM.gov — 155mm Towed Howitzer Systems
  category: missiles_ad (wrong category — probably should be artillery)
  value: $240M → $240,000,000 USD
  deadline: null → dl_days: null → status: "open"
  lean: "maybe"

Note: Tender #2 category "missiles_ad" means it matched against
Spike, Barak-8, and Anti-tank products instead of artillery —
likely a crawler classification error.
```

**Code flow:**
1. `best_pct` from all product matches → determines verdict:
   - `≥80` → **go** — "Strong fit, pursue."
   - `≥55` → **maybe** — "Partial fit — qualify before committing."
   - `<55` → **pass** — "Weak fit — monitor only."
2. `deadline_date - today` → `dl_days`
   - `<0` → closed, `0-7` → closing, `>7` → open
3. `value_num × FX[currency]` → `value_usd`
4. Writes `srv_tenders` + clears and rewrites `srv_tender_matches`

---

### Partnership Processing

**File:** `services/domain_pipeline.py` → `process_partnership()`

**What it does:** Tags partnerships by relevance to KSSL's product lines and generates competitive meaning analysis.

**Real example from today:**

```
Partnership #1: NIBE Limited ← Sig Sauer (license agreement)

  detected_lines: ["small_arms", "missiles_ad"]
  → kssl_relevance: CORE (directly competes with KSSL's small arms line)

  partner_kind: null → "Domestic tie (Sig Sauer) — lower sanctions risk"

  meaning:
    "Threat: NIBE Limited gains small_arms, missiles_ad capability
     via Sig Sauer.
     Opening: KSSL differentiators remain indigenous IP, forging
     scale and trials-maturity.
     Dependency: Domestic tie (Sig Sauer) — lower sanctions risk."

Partnership #2: Solar Industries India ← EDGE Group (MoU)

  detected_lines: ["uav"]
  → kssl_relevance: CORE (directly competes with KSSL's UAV line)

  partner_kind: null → "Domestic tie (EDGE Group) — lower sanctions risk"

  meaning: Same threat/opening/dependency structure.
```

**Code flow:**
1. Resolve competitor via S-05 (same function)
2. Check `detected_lines`:
   - Present → **CORE** (competes on a KSSL product line)
   - No lines but has `partner_kind` → **ADJACENT** (related field)
   - Neither → **context** (background awareness)
3. `partner_kind == "Foreign OEM"` → dependency text mentions IP held by foreign entity, higher sanctions risk. Otherwise → "Domestic tie — lower sanctions risk."
4. Build meaning string with three-part structure: Threat / Opening / Dependency

---

### Geo Footprint Processing

**File:** `services/domain_pipeline.py` → `process_geo()`

**What it does:** Tags geographic deployment records by confidence level.

**Real example from today:**

```
Geo #1: Solar Industries → Armenia, product: Nagastra-1/2/3
  confidence: "high" → provenance: "sourced" (verified deployment)
  stage: "Contracted" — actually sold, not just offered

Geo #2: Solar Industries → Armenia (same article, different product view)
  confidence: "high" → provenance: "sourced"
```

**Code flow:**
1. `confidence == "high"` or `"medium"` → `provenance: "sourced"` (verified record)
2. `confidence == "low"` → `provenance: "estimate"` (rumoured)
3. All other fields are pass-through — no computation

---

### Innovation Processing

**File:** `services/domain_pipeline.py` → `process_innovation()`

**What it does:** Tags tech developments by how far ahead or behind they are vs KSSL, and generates impact analysis.

**Real example from today:**

```
Innovation #1: Rheinmetall demonstrates ramjet 155mm projectile (70km range)
  competitor_id: "RHEIN" → gap_vs_kssl: "behind"
  (KSSL doesn't have a ramjet 155mm round)

  maturity: "test" — still in trials, not fielded yet
  horizon: "2027-2028" — 1-2 years from operational

  impact: "If fielded on 2027-2028, this shifts the artillery benchmark
           KSSL is measured against."

  action: "Assess KSSL's roadmap in artillery against this development
           and brief R&D."

Innovation #2: Loitering munitions mature as one-way attack drones
  competitor_id: null → gap_vs_kssl: "parity"
  (Industry-wide trend, KSSL is also developing loitering munitions)

  maturity: "ioc" — initial operational capability
  impact + action: Same template structure
```

**Code flow:**
1. Has `competitor_id` → gap = **behind** (competitor specific, KSSL not mentioned)
2. No `competitor_id` → gap = **parity** (industry-wide, KSSL tracks it)
3. Build `impact` using `horizon_hint` + `tech_domain`
4. Build `action` using `tech_domain` → "Assess KSSL's roadmap and brief R&D"

---

### S-25: Mallory Chat (RAG)

**File:** `services/assistant.py` → `answer()`

**What it does:** A Retrieval-Augmented Generation chat system. When the frontend user asks Mallory a question, it scopes the context to whatever panel they're viewing and asks the LLM to answer ONLY from that context.

**How it works:**

```
Frontend user opens a signal card → clicks "Ask Mallory" → types a question

1. Scope context by panel:
   Signal panel  → loads SrvSignalDetail (facts, what_text, why_text, lens_reads, actions)
   Tender panel  → loads SrvTender + all SrvTenderMatch (product names + fit %)
   Competitor    → loads SrvCompetitorSynthesis (thesis, vulnerabilities, predictions)
                  + 6 latest SrvPartnership
   Overview      → top 8 SrvSignal by rank

2. System prompt:
   "You are Mallory, KSSL's competitive-intelligence analyst. Answer ONLY
    from the provided context, always framed as what it means for KSSL.
    If the answer is not in the context, say you don't have that data.
    Be concise and concrete."

3. Context + question → LLM → answer

4. Returns: { answer, scope, sources[] }
```

**Example — if a user asks about Signal #21 (KNDS Nigeria):**

The context would include:
```
SIGNAL: KNDS remporte une commande de CAESAR 6x6 pour le Nigeria
Direction vs KSSL: threat. This strengthens KNDS...
Facts: Company=KNDS; Country=Nigeria; Domain=artillery
Why it matters: Read against KSSL's portfolio...
[BENCHMARK] This strengthens KNDS on a line KSSL contests...
[POLICY/OFFSET] Indigenous-content and offset rules remain KSSL's lever...
[Counter action] Brief KSSL BD on a positioning response within the week.
[Benchmark action] Compare the named capability against KSSL's equivalent product.
```

If the user asks "Should KSSL counter-bid?" — Mallory would answer from the actions/analysis in the detail view. If the user asks "What's the weather in Nigeria?" — Mallory says "I don't have that data."

---

### S-26: CEO Report

**File:** `services/assistant.py` → `ceo_report()`

**What it does:** Generates a structured executive brief from the top threats, go-tenders, and innovations.

**Code flow:**
1. Query top 5 threats: `SELECT * FROM srv_signals WHERE dir='threat' ORDER BY rank LIMIT 5`
2. Query top 5 go tenders: `SELECT * FROM srv_tenders WHERE lean='go' LIMIT 5`
3. Query top 5 innovations: `SELECT * FROM srv_innovation LIMIT 5`
4. Build context string from `sowhat`, `lean_text`, `gap_vs_kssl`, `impact`
5. LLM generates 3-sentence executive summary
6. Returns 4 sections:
   - Executive summary (LLM generated)
   - Top competitive threats (5 rows with title + sowhat)
   - Tender priorities (5 rows with title + lean_text)
   - Technology watch (5 rows with title + gap + impact)

**Real data available today for the report:**
- 27 threat signals across 3 pillars
- 2 tenders (both "maybe", no "go" yet — due to missing requirement_fields)
- 2 innovations (Rheinmetall ramjet, loitering munitions)

---

### Auto-Processor

**File:** `services/auto_processor.py`

**What it does:** Background scheduler that runs the entire pipeline every 5 minutes automatically.

**Code flow:**
1. At app startup, spawns an `asyncio` task
2. Sleeps 300 seconds (first run at 5 min mark)
3. Gets fresh DB session, calls `process_pending()` — same function the manual endpoint uses
4. Logs: `Auto-pipeline: signals=6, tenders=3, partnerships=3, geo=2, innovation=2`
5. Commits, closes session, sleeps 300 more seconds
6. Repeats forever

**Why 5 minutes?** Balances freshness against compute cost. In production with high crawler volume, this could be reduced to 60 seconds or made event-driven (triggered by the crawler after a batch).

---

## 6. Critical crawler fields — What feeds the engine

These are the fields that directly power the intelligence engine. If any are missing, the output degrades.

| Field | Record type | Criticality | What happens if missing |
|---|---|---|---|
| `event_summary` | signal | ⚠️ CRITICAL | No classification, no enrichment, no sowhat. Signal becomes useless. |
| `stream` | signal | ⚠️ CRITICAL | No pillar routing. Signal goes nowhere. |
| `competitor_id` | signal | ⚠️ HIGH | Falls back to alias scanning in main_text — works but less reliable. |
| `main_text` | document | ⚠️ HIGH | Entity resolution has no fallback text to scan for aliases. |
| `requirement_fields` | tender | ⚠️ CRITICAL | All matches default to 55%. No spec comparison. **This is broken today.** |
| `category_hint` | tender | ⚠️ CRITICAL | Can't find KSSL products to match against. |
| `deadline_date` | tender | ⚠️ HIGH | No days-remaining or open/closing/closed status. |
| `value_num` + `value_currency` | tender | MEDIUM | No USD conversion, can't sort by value. |
| `published_at` | signal | MEDIUM | No "ago" display, ranking defaults to insertion order. |
| `detected_lines` | partnership | MEDIUM | kssl_relevance defaults to "context" instead of CORE. |
| `partner_kind` | partnership | LOW | Meaning text uses generic template instead of Foreign OEM analysis. |
| `confidence` | geo | LOW | Provenance tag defaults — no impact on computation. |
| `tech_domain` | signal | LOW | Missing from facts bar — cosmetic only. |
| `deal_value_raw` | signal | LOW | Missing from facts bar — cosmetic only. |

**Today's biggest gap:** `requirement_fields` is empty on both tenders. The crawler is extracting tender text into `requirement_text` (the full RFP body) but not parsing it into structured `[{label, value}]` pairs. This means the scoring engine can't do spec matching — every product gets 55%.

---

## 7. End-to-end trace — One real signal from crawler to frontend

### One real signal: KNDS wins Nigeria CAESAR order

```
┌─────────────────────────────────────────────────────────────────────────┐
│ CRAWLER INPUT                                                            │
│                                                                          │
│ POST /ingest/v1/competitive_signal                                       │
│ {                                                                        │
│   "document": {                                                          │
│     "url": "https://defensenews.com/...knds-caesar-nigeria",             │
│     "main_text": "KNDS remporte une commande de CAESAR 6x6 pour le       │
│        Nigeria... 18 systèmes d'artillerie CAESAR 6x6 de 155 mm...",     │
│     "title": "KNDS remporte une commande de CAESAR pour le Nigeria",     │
│     "source_id": "DEFNEWS",                                              │
│     "screenshot": { "storage_path": "s3://mallory-raw/shot/25ca33.png" } │
│   },                                                                     │
│   "record": {                                                            │
│     "stream": "competitive",                                             │
│     "competitor_id": "KNDS",                                             │
│     "event_summary": "KNDS wins CAESAR artillery contract in Nigeria",   │
│     "detected_country": "Nigeria",                                       │
│     "tech_domain": "artillery"                                           │
│   }                                                                      │
│ }                                                                        │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ stg_documents                                                            │
│   id: "doc_29fe..." (sha1 of URL)                                        │
│   main_text: "KNDS remporte une commande..."                             │
│   title: "KNDS remporte une commande..."                                 │
│   screenshot: {storage_path: "s3://mallory-raw/shot/25ca33.png"}         │
│                                                                          │
│ stg_signals                                                              │
│   id: 21                                                                 │
│   document_id: "doc_29fe..."                                             │
│   stream: "competitive"                                                  │
│   competitor_id: "KNDS"                                                  │
│   event_summary: "KNDS wins CAESAR artillery contract in Nigeria"        │
│   detected_country: "Nigeria"                                            │
│   tech_domain: "artillery"                                               │
│   proc_status: "received"                                                │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                    [AUTO-PIPELINE: 5 min interval]
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ S-05: ENTITY RESOLUTION                                                  │
│                                                                          │
│ competitor_id="KNDS" → SELECT FROM ref_competitors WHERE id='KNDS'       │
│ → FOUND: id=KNDS, name="KNDS (Nexter / KMW)", aliases=["Nexter"],       │
│   hq_country="France/Germany", threat_level="threat", priority="P1"      │
│                                                                          │
│ resolved_competitor_id = "KNDS" ✓                                        │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ S-07: CLASSIFICATION                                                     │
│                                                                          │
│ event_summary: "KNDS wins CAESAR artillery contract in Nigeria"          │
│                                                                          │
│ Keyword scan: "wins" → THREAT word ✓                                     │
│ Stream: competitive → lens = BENCHMARK                                   │
│ Tags: ["threat"]                                                         │
│                                                                          │
│ dir = "threat"                                                           │
│ lens = "BENCHMARK"                                                       │
│ tags = ["threat"]                                                        │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ S-09: ENRICHMENT                                                         │
│                                                                          │
│ facts = [                                                                │
│   ["Company", "KNDS (Nexter / KMW)"],                                    │
│   ["Domain", "artillery"],                                               │
│   ["Country", "Nigeria"]                                                 │
│ ]                                                                        │
│                                                                          │
│ dir = "threat" → threat template:                                        │
│   sowhat = "This strengthens KNDS on a line KSSL contests —              │
│             expect direct pressure on KSSL's bids and pricing."          │
│                                                                          │
│   what_text = full event_summary                                         │
│                                                                          │
│   why_text = "Read against KSSL's portfolio, this moves the              │
│               competitive balance in competitive. This strengthens        │
│               KNDS on a line KSSL contests..."                           │
│                                                                          │
│   lens_reads = [                                                         │
│     ["BENCHMARK", "This strengthens KNDS on a line..."]                  │
│     ["POLICY/OFFSET", "Indigenous-content and offset rules remain        │
│       KSSL's structural lever here."]                                    │
│   ]                                                                      │
│                                                                          │
│   actions = [                                                            │
│     ["Counter", "Brief KSSL BD on a positioning response within          │
│       the week."],                                                       │
│     ["Benchmark", "Compare the named capability against KSSL's           │
│       equivalent product."]                                              │
│   ]                                                                      │
│                                                                          │
│   suggest = [                                                            │
│     "What does this mean for KSSL?",                                     │
│     "Who else is affected?",                                             │
│     "Show the head-to-head."                                             │
│   ]                                                                      │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ S-10: RANKING                                                            │
│                                                                          │
│ Competitive pillar: 27 threats, 41 watch, 1 fav                          │
│ Sort: dir_weight(3→threat, 2→watch, 1→fav) + published_at(desc)          │
│                                                                          │
│ Signal #21 rank: 5 (5th threat by recency among 27 threats)              │
│ rank_group: "Priority — Threats"                                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ PUBLISHED TO SERVING                                                     │
│                                                                          │
│ srv_signals (id=21):                                                     │
│   pillar: "competitive"                                                  │
│   dir: "threat"                                                          │
│   rank: 5                                                                │
│   rank_group: "Priority — Threats"                                       │
│   title: "KNDS remporte une commande..."                                 │
│   meta: "artillery · KNDS (Nexter / KMW)"                                │
│   company: "KNDS (Nexter / KMW)"                                         │
│   lens: "BENCHMARK"                                                      │
│   sowhat: "This strengthens KNDS..."                                     │
│   tags: ["threat"]                                                       │
│   ago_display: "5d ago"                                                  │
│   source_url: "https://defensenews.com/..."                              │
│                                                                          │
│ srv_signal_details (signal_id=21):                                       │
│   title: "KNDS remporte une commande..."                                 │
│   facts: [["Company","KNDS"],["Domain","artillery"],["Country","Nigeria"]]│
│   what_text: "KNDS remporte une commande..."                             │
│   why_text: "Read against KSSL's portfolio..."                           │
│   lens_reads: [["BENCHMARK","..."],["POLICY/OFFSET","..."]]              │
│   actions: [["Counter","..."],["Benchmark","..."]]                       │
│   suggest: ["What does this mean...","Who else...","Show head-to-head"]  │
│                                                                          │
│ srv_overview_metrics: recalculated — threats: 27, total: 69              │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ FRONTEND READS                                                           │
│                                                                          │
│ GET /api/v1/signals?pillar=competitive                                   │
│                                                                          │
│ Response:                                                                │
│ {                                                                        │
│   items: [                                                               │
│     { id: 21, pillar: "competitive", dir: "threat", rank: 5,             │
│       title: "KNDS remporte une commande de CAESAR 6x6...",              │
│       meta: "artillery · KNDS",                                          │
│       sowhat: "This strengthens KNDS on a line KSSL contests...",        │
│       lens: "BENCHMARK",                                                 │
│       tags: ["threat"],                                                  │
│       ago_display: "5d ago",                                             │
│       company: "KNDS (Nexter / KMW)"                                     │
│     },                                                                   │
│     ... more signals ...                                                 │
│   ],                                                                     │
│   page: 1, size: 20, total: 69                                           │
│ }                                                                        │
│                                                                          │
│ Frontend renders:                                                        │
│ ┌──────────────────────────────────────────┐                             │
│ │ ⚠ THREAT  ·  BENCHMARK  ·  5d ago        │                             │
│ │ KNDS remporte une commande de CAESAR...   │                             │
│ │ artillery · KNDS (Nexter / KMW)           │                             │
│ │                                            │                             │
│ │ This strengthens KNDS on a line KSSL      │                             │
│ │ contests — expect direct pressure on      │                             │
│ │ KSSL's bids and pricing.                  │                             │
│ └──────────────────────────────────────────┘                             │
│                                                                          │
│ Click → GET /api/v1/signals/21/detail                                   │
│                                                                          │
│ ┌──────────────────────────────────────────┐                             │
│ │ Facts:                                    │                             │
│ │   Company: KNDS (Nexter / KMW)            │                             │
│ │   Domain: artillery                       │                             │
│ │   Country: Nigeria                        │                             │
│ │                                            │                             │
│ │ What happened:                             │                             │
│ │   KNDS remporte une commande...            │                             │
│ │                                            │                             │
│ │ Why it matters vs KSSL:                    │                             │
│ │   Read against KSSL's portfolio...         │                             │
│ │                                            │                             │
│ │ Analysis reads:                            │                             │
│ │   BENCHMARK: This strengthens KNDS...      │                             │
│ │   POLICY/OFFSET: Indigenous-content...     │                             │
│ │                                            │                             │
│ │ Actions:                                   │                             │
│ │   Counter: Brief KSSL BD...                │                             │
│ │   Benchmark: Compare capability...         │                             │
│ │                                            │                             │
│ │ Suggested follow-ups:                      │                             │
│ │   What does this mean for KSSL?            │                             │
│ │   Who else is affected?                    │                             │
│ │   Show the head-to-head.                   │                             │
│ └──────────────────────────────────────────┘                             │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Summary: What creates the intelligence

1. **S-05** maps a raw name to a known competitor profile from the seed
2. **S-07** reads the event text and tags every signal as threat/watch/fav vs KSSL
3. **S-09** generates actionable analysis — why it matters, what to do, what to investigate
4. **S-10** prioritizes — threats first, newest first
5. **S-11** counts everything so the frontend shows live numbers
6. **S-12/S-13** match tenders against KSSL product specs and decide go/maybe/pass
7. **Partnership processing** tags by relevance to KSSL's specific product lines
8. **Innovation processing** measures how far ahead or behind KSSL is
9. **S-25** lets users ask natural-language questions grounded in real data
10. **S-26** generates executive briefs from top signals
11. **Auto-processor** keeps everything fresh every 5 minutes

**The intelligence isn't magic** — it's a deterministic pipeline where each stage transforms raw crawler data into increasingly structured, ranked, and framed analysis. The LLM (when enabled) adds depth and context understanding; without it, keyword-based rules still produce meaningful output. The entire system is designed so that if the LLM is unavailable, the pipeline still produces useful intelligence — just less nuanced.
