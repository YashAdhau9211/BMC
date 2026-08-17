# DFS — 04 · Data Model, Provenance, Evaluation, Failure Handling, Roadmap

---

## 1. Data model

Two layers, hard-separated (see `03-conflict-and-belief.md` §7). Postgres is sufficient for v1 — the only "graph" is three edge tables and the only "vectors" are a coverage array and text fingerprints.

### 1.1 Evidence layer — append-only, never mutated

```sql
source(
  source_id PK, display_name, source_type, publisher_org, first_seen,
  practice JSONB,                        -- cites_primary, named_bylines, corrections, ...
  coverage JSONB,                        -- domain → [0,1], recomputed monthly (COVERAGE only)
  channel_verified BOOL, verified_at,
  adversarial_flag BOOL, under_review BOOL
)

source_edge(from_source, to_source, kind, weight, observed_at)
  -- kind ∈ OWNS | SYNDICATES_TO | CITES | SHARES_BYLINE      ← the entire "knowledge graph"

document(
  doc_id PK,                             -- content hash; also the dedup key
  source_id FK, url, title, language,
  published_at, retrieved_at,            -- MUST stay distinct (copy-chain direction)
  raw BYTEA,                             -- verbatim; span validation reads this
  handling JSONB, collection_ctx JSONB,
  ingested_at
)

provenance_feature(
  doc_id FK, simhash BIGINT, minhash_sig BYTEA,
  numeric_fingerprint TEXT[],            -- value+unit+rounding tuples
  citations JSONB, wire_credit, bylines TEXT[], publisher_resolved
)

claim(                                   -- THE ATOM. Append-only. Never updated.
  claim_id PK, doc_id FK, source_id FK,
  span_start INT, span_end INT,          -- validated against document.raw
  claim_type,                            -- PROC | CAP | EVENT | OOB | DEPLOY | INTENT | ATTRIB
  domain,                                -- selects the competence cell with claim_type
  entity_id, variant_id, entity_confidence, variant_confidence,
  attribute, quantity_type,
  value_num NUMERIC, value_text, unit, granularity NUMERIC,
  qualifiers JSONB,                      -- each key: {value, state: KNOWN|UNKNOWN}
  hypotheses JSONB,                      -- flagged imputations; EXCLUDED from contradiction tests
  qualification_completeness NUMERIC,
  stance, assertion_strength,            -- hedging: "reportedly", "up to"
  asserted_valid_from, asserted_valid_to,
  report_date, extracted_at,
  extractor_version, validator_passed BOOL,
  status                                 -- ACTIVE | PARKED_ENTITY | QUARANTINED | WITHDRAWN
)

provenance_edge(from_claim, to_claim, rho NUMERIC, method, detected_at)
  -- method ∈ WIRE | NEAR_DUP | CITATION | NUM_FINGERPRINT | BYLINE | OWNER | TIME_LAG

outcome(                                 -- ground truth ledger; drives ALL reputation
  outcome_id PK, claim_id FK, resolver, verdict,   -- CORRECT | WRONG | FABRICATED | UNRESOLVED
  novelty_weight NUMERIC,                          -- 1.0 first-mover, 0.25 echo
  resolved_at, lag_days, adjudicator, note
)
```

### 1.2 Reputation

```sql
source_cell(
  source_id, domain, claim_type,         -- PK triple
  alpha NUMERIC, beta NUMERIC,           -- Beta posterior
  n_asserted INT, n_resolved INT, resolution_rate NUMERIC,
  prior_m NUMERIC, prior_kappa NUMERIC,
  mean NUMERIC, lcb10 NUMERIC,           -- lcb10 = L, the ONLY number that reaches belief math
  capped_by_resolution_rate BOOL, updated_at
)

source_tier_view(source_id, tier, since, basis_cell)   -- DERIVED. Never an input to math.
```

### 1.3 Belief layer — derived, versioned, recomputable

```sql
fact(
  fact_id PK, entity_id, variant_id, attribute, quantity_type,
  qualifier_signature TEXT               -- the grouping key
)

fact_version(                            -- bitemporal
  fact_version_id PK, fact_id FK,
  value_num, value_text, unit, granularity,
  status,                                -- corroborated | single_source | consensus_with_dissent
                                         -- | disputed | not_comparable | quarantined_only | refuted
  confidence NUMERIC,                    -- deterministic P from G5
  k_clusters INT, n_eff NUMERIC, raw_source_count INT,
  valid_from, valid_to,                  -- VALID time  (when true of the world)
  record_from, record_to,                -- RECORD time (when we believed it)
  supersedes FK, supersession_kind,      -- CHANGE | CORRECTION | UNDETERMINED
  algorithm_version, config_hash,        -- ← makes every version reproducible
  decision_outcome, reason_codes TEXT[], computed_at,
  reviewed_by, reviewed_at
)

fact_evidence(fact_version_id FK, claim_id FK, role, cluster_id, weight)
  -- role ∈ SUPPORTING | DISSENTING | QUARANTINED | CIRCULAR

provenance_cluster(cluster_id PK, fact_version_id FK, root_claim_id, member_claims UUID[],
                   rho_mean NUMERIC, method_summary)

review_task(task_id PK, subject_type, subject_id, impact, uncertainty,
            priority NUMERIC,            -- impact × uncertainty, NOT FIFO
            sla_due_at, assigned_to, state, resolution JSONB)

audit_log(actor, action, subject, before JSONB, after JSONB, at)
```

**Invariant enforced at the DB level:** `claim` has no UPDATE grant except `status`. Every other correction is a new row.

---

## 2. Provenance model

Three questions must be answerable from storage alone, with no reconstruction:

| Question | Answered by |
|---|---|
| **Why do we believe this?** | `fact_version → fact_evidence → claim → span → document.raw` — down to the exact bytes of the original text. |
| **What did we believe on date D, and why?** | `fact_version` record-time range at D, plus its frozen `algorithm_version` + `config_hash`. |
| **Who was first?** | `provenance_edge` DAG root + `published_at` ordering. This is what makes scoop credit computable at all. |

**Reproducibility contract (NFR-1).** Every `fact_version` stores the algorithm version and config hash that produced it. Replaying that version's evidence set under that version+hash must produce a bit-identical result. This is the regression gate for every algorithm change, and it is why no LLM may touch belief computation.

**Backward cascade (FR-11).** When a claim is withdrawn, a source is flagged adversarial, or a retraction lands:

```
affected_claims  ← claim ∪ transitive closure over provenance_edge (copies inherit the taint)
affected_facts   ← facts whose latest fact_version cites any affected claim
→ recompute those facts, write new fact_versions with reason_code CASCADE_RECOMPUTE
→ diff old vs new confidence; any ACCEPT→below-threshold flip raises a P1 alert
→ notify every downstream consumer that read the affected fact within the retention window
```

Consumer notification is not optional. A KB that silently corrects itself has told its consumers nothing.

---

## 3. Evaluation metrics

**Calibration is the headline metric, not accuracy.**

### 3.1 Belief quality

| Metric | Definition | Target v1 |
|---|---|---|
| **ECE** | Expected calibration error over resolved claims, 10 bins | < 0.07 |
| **Brier score** | Mean squared error of `P` vs resolved outcome | < 0.15 |
| **Bin honesty** | Empirical accuracy within each confidence decile | Every bin within ±0.10 of its nominal |
| **Over-confidence rate** | Fraction of `P ≥ 0.85` ACCEPTs later refuted | < 3% |
| **Under-confidence rate** | Fraction of `P < 0.55` HOLDs later confirmed | Tracked; falling trend |

Calibration must be reported **stratified by claim type**, because `CAP` claims resolve rarely and pooled calibration would hide the exact segment that matters most.

### 3.2 Structural metrics (these test the design's core claims)

| Metric | Definition | Target |
|---|---|---|
| **Echo collapse ratio** | On known wire-syndicated stories: `n_eff / raw_source_count` | ≤ 0.10 (40 copies → ≈1) |
| **False-conflict rate** | Fraction of flagged conflicts that adjudication finds were qualifier/unit/entity mismatches | < 15% (this is the type-1–4 dissolution working) |
| **Scoop latency** | Median time from a HOLD claim's first appearance to promotion after independent confirmation | < 72h |
| **Scoop recovery rate** | Fraction of later-confirmed claims that were held (not lost) at first appearance | > 95% |
| **Span validity** | Claims whose spans and numeric tokens verify against `document.raw` | 100% (hard gate) |
| **Entity resolution F1** | Against a curated variant-labelled test set | > 0.92 entity, > 0.85 variant |
| **Cell coverage** | Fraction of active (source × domain × claim-type) cells with `n_resolved ≥ 10` | Tracked; the honest measure of how much of the model is data vs prior |

### 3.3 Operational

| Metric | Target |
|---|---|
| Analyst queue drain vs SLA (critical 4h / sensitive 24h) | > 95% within SLA |
| Analyst overturn rate of ACCEPT decisions | < 5% (higher ⇒ thresholds are wrong) |
| Analyst agreement with PROVISIONAL/REVIEW routing | > 80% (lower ⇒ routing is noise) |
| G1 reject sample false-negative rate | < 1% of sampled rejects are genuinely defence-relevant |

---

## 4. Test strategy

| Layer | Approach |
|---|---|
| **Golden claim set** | ~500 hand-adjudicated claims with known outcomes, spanning all 7 claim types and all 7 domains, including deliberately under-qualified ones. Regression baseline for extraction, resolution, and calibration. |
| **Replay harness** | Freeze the evidence layer; recompute the entire belief layer under a candidate algorithm; **diff every fact**. Any change ships with its diff report. This is the primary safety net and the reason NFR-1 exists. |
| **Adversarial suite** — synthetic, run every build | ① *Reputation farm*: a simulated source builds 18 months of correct `PROC` claims, then asserts a false `CAP` claim. **Pass = the false claim does not reach ACCEPT.** ② *Sockpuppet ring*: 12 sources publish the same claim within 3h with no citations. **Pass = `n_eff < 2` and `campaign_suspected` raised.** ③ *Unit injection*: nmi/km, mi/km, ft/m contamination. **Pass = classified as type-1 artifact, never as conflict.** ④ *Variant confusion*: Su-35 vs Su-35S figures mixed. **Pass = split into two facts, not disputed.** ⑤ *Wire cascade*: one story, 40 copies. **Pass = `n_eff ≤ 1.1`.** ⑥ *Stale authority*: a 1998 official manual vs 2024 test data on an upgraded system. **Pass = versioned, not disputed.** ⑦ *Precision laundering*: "≈200" re-reported as "204". **Pass = granularity capped at origin, no weight gain.** |
| **Property tests** | Belief computation determinism (same input+hash → identical output) · append-only invariant (no UPDATE on `claim` outside `status`) · every API response contains `status` · no code path computes a mean of asserted values. |
| **Shadow mode** | Run 4 weeks before any auto-ACCEPT is enabled. All decisions computed and logged, none published. Analysts adjudicate a sample; compare to the system's decision. Gate to production: overturn rate < 5%, ECE < 0.07. |
| **Red team** | Quarterly human exercise: given the published thresholds and source model, get a false claim to ACCEPT. Every success becomes an adversarial-suite case. |

---

## 5. Failure handling

| Failure | Behaviour |
|---|---|
| **Extractor (LLM) unavailable** | Documents queue at G1-passed. **No degraded extraction path** — a worse extractor writing to an append-only store is permanent damage. Alert on queue depth. |
| **Extractor validator failure rate spikes** | Circuit-break at > 5% span-validation failure over 1000 claims: halt extraction, alert, quarantine the batch pending model/prompt review. Silent extraction drift is the most dangerous degradation in the system. |
| **Backpressure / queue overflow** | Shed load by **deferring**, never by relaxing thresholds. Under load the system produces fewer ACCEPTs, never lower-quality ones. |
| **Source compromised / exposed as adversarial** | `adversarial_flag`, backward cascade (§2), consumer notification, all its claims → `QUARANTINED` in belief while remaining in evidence. |
| **Retraction from a source** | New claim atom with `stance = retraction`; cascade over the derivation DAG; copies of the retracted claim are down-weighted, not deleted. |
| **Poisoned reputation cell** (bad outcomes ingested) | `source_cell` keeps an append-only update journal; roll back to a prior `(alpha, beta)` snapshot and replay the ledger from a chosen point. |
| **Config / threshold change** | Versioned; requires a replay diff report before merge; old `fact_version`s keep their original `config_hash` and are never retroactively rewritten. |
| **Entity registry error** (bad alias merges two systems) | Alias changes trigger targeted recompute of all facts touching either entity; alias edits are audit-logged with an actor. |
| **Analyst queue starvation** | SLA breach alerts; if `critical` queue exceeds SLA, auto-ACCEPT for that impact class is **disabled** (fail closed, not open). |
| **Clock skew / bad `published_at`** | Timestamps failing sanity checks are marked untrusted; affected pairs fall back to citation and text-similarity signals for ρ, never to time ordering alone. |
| **Downstream consumer ignores `status`** | Contract test in the consumer's CI; the API has no value-only path to ignore (FR-12). |

**Fail-closed principle, stated once:** every degraded mode in this table produces *less output*, never *less-verified output*.

---

## 6. Implementation roadmap

Each milestone has a numeric exit gate. Nothing ships to the next milestone without it.

### M0 — Skeleton (weeks 1–4) · fully deterministic, no LLM
Schema (evidence + belief) · G0 intake with fingerprints, citations, dual timestamps · source registry + three edge tables + hand-set type priors · deterministic normaliser (units, conversion-artifact detector, plausibility bands) · entity/alias authority file for a **single pilot domain** (recommend `AEROSPACE`, best resolver coverage) · G1 relevance.
**Exit:** 10k documents ingested; 100% dual-timestamp and fingerprint capture; unit-artifact detector catches 100% of an injected test set.

### M1 — Claims & echo (weeks 5–10)
S1 caged extraction + span validator · qualifier vector with UNKNOWN states for 10 pilot attributes · S2 entity/variant resolution · S3 provenance clustering + one-vote-per-cluster · rule-based belief view with **static type priors only** (no reputation yet).
**Exit:** span validity 100% · echo collapse ratio ≤ 0.10 on a labelled wire-cascade set · entity F1 > 0.90 · false-conflict rate measured (no target yet, this is the baseline).

### M2 — Reputation (weeks 11–16)
Outcome ledger + automated resolvers (contract databases, official confirmations) · `source_cell` Beta posteriors + hierarchical priors + resolution-rate cap + novelty weighting · tier view with hysteresis · analyst review queue with impact × uncertainty ordering.
**Exit:** ≥ 30% of active cells at `n_resolved ≥ 10` · analyst queue drains within SLA · tier flapping < 2 transitions/source/quarter.

### M3 — Belief & conflict (weeks 17–24)
Full G5 formula with `Λmax` cap and impact-forced review · conflict taxonomy tests 1–7 in order · consensus-with-dissent + disputed states · bitemporal versioning + supersession · replay harness · calibration dashboard.
**Exit:** **ECE < 0.07** · Brier < 0.15 · overturn rate < 5% · full replay completes < 6h · adversarial suite cases ①–⑦ all pass.

### M4 — Shadow → production (weeks 25–30)
4-week shadow run · threshold calibration against analyst adjudications · consumer API + contract tests · cascade/notification pipeline.
**Exit:** shadow overturn < 5% · zero value-only read paths in any consumer · cascade drill executed end-to-end (flag a test source adversarial, verify every dependent fact recomputes and every consumer is notified).

### M5+ — Learned components (post-launch, each independently gated on beating the deterministic baseline in replay)
Fitted partial-pooling model replacing the fixed prior blend · learned pairwise ρ from longitudinal co-movement · coverage classifier replacing hand-tagged domains · campaign detection beyond the synchrony heuristic · learned qualifier-sensitivity tables · multilingual extraction expansion.

**Sequencing rationale:** reputation (M2) is deliberately *after* echo detection (M1), because reputation computed over uncollapsed copies trains on amplification rather than accuracy — and a reputation model poisoned at the start is far more expensive to unwind than one built late.
