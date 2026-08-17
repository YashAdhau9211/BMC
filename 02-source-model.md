# DFS — 02 · Source Profile, Scoring & Vectorisation

---

## 1. The core correction: separate COVERAGE from COMPETENCE

The proposed Source Vector —

```
Defence 0.95 · Energy 0.30 · Technology 0.85 · Aerospace 0.92 · Cyber 0.70 · Procurement 0.88
```

— conflates two different things and encodes both dishonestly.

- **Where do the numbers come from?** If hand-assigned, this is a spreadsheet with false precision: two decimals claim ~1% resolution, and no human assigner can reproducibly distinguish 0.88 from 0.92 (inter-rater agreement on such tasks is ±0.1 at best). If learned, you need `(source, claim, resolved outcome)` triples — roughly **n ≈ 50–100 resolved claims per cell** before a 0.95 source is statistically separable from a 0.85 one. Most sources will never reach that in a niche. **So for most of the matrix, the prior IS the system** — and it must therefore be designed honestly rather than hidden behind decimals.
- **A scalar cannot express sample size.** 3/3 correct and 285/300 correct both read as "≈0.95". They are radically different objects, and a point score ranks them *backwards*.

**Split into two representations:**

| | **Coverage vector** | **Competence posteriors** |
|---|---|---|
| Answers | *What does this source write about?* | *Is this source right, here?* |
| Derived from | Corpus topic classification / embeddings | The outcome ledger, counted |
| Shape | Dense `[0,1]` vector over ~7 domains | Sparse grid of Beta posteriors over (domain × claim-type) |
| Job | Selects **which competence cells apply**; routing; discovery | All weighting math |
| May set trust? | **No.** A source can publish 10,000 aerospace articles and be wrong in all of them. | Yes — it is the only thing that may. |

This is the answer to "what is the correct representation": the user's vector is real and useful, but it is a *coverage* vector, not a trust vector.

---

## 2. Competence: Beta-Binomial per (source × domain × claim-type)

### 2.1 The cut

Domain alone is the wrong cut — the brief's own example proves it: a source excellent at *procurement contract awarded* can be terrible at *radar detection range*, because the first is checkable against SAM.gov / TED / DSCA within weeks and the second is physics laundered from vendor marketing.

**Claim-type taxonomy (7):**

| Code | Claim type | Typically resolvable? |
|---|---|---|
| `PROC` | Contract / procurement event | Yes, weeks |
| `CAP` | Capability / performance specification | **Rarely** |
| `EVENT` | Event occurrence (strike, test, delivery, incident) | Often, days–weeks |
| `OOB` | Inventory / order-of-battle quantity | Partially |
| `DEPLOY` | Deployment / location / basing | Often, imagery |
| `INTENT` | Policy / intent / plan | Weakly, months–years |
| `ATTRIB` | Attribution (who did it, who built it) | Rarely |

**Domain taxonomy (7):** `DEFENCE_LAND · AEROSPACE · NAVAL · C4ISR/CYBER · SPACE · PROCUREMENT/INDUSTRY · GEOPOLITICS`.

7 × 7 = 49 cells per source, most empty. Sparsity is handled by pooling, not by abandoning the cut.

### 2.2 The posterior

```
prior      Beta(α₀, β₀)  with  α₀ = κ₀·m,  β₀ = κ₀·(1−m),  κ₀ = 8
update     observe k correct of n resolved  →  Beta(α₀ + k, β₀ + n − k)
score      L = LCB₁₀ = BetaPPF(0.10, α, β)          ← the number used in all weighting
display    (L, mean, n, resolution_rate)             ← NFR-5: never a naked decimal
```

Closed-form, incremental, auditable, no training pipeline.

**Why the lower bound and not the mean:**

| Record | Posterior | Mean | **L = LCB₁₀** |
|---|---|---|---|
| 3 of 3 correct (flat prior) | Beta(4, 1) | 0.80 | **0.56** |
| 285 of 300 correct | Beta(286, 16) | 0.947 | **0.93** |

The LCB ranks the seasoned source above the lucky newcomer. The point score does not. This single choice also solves cold start for free (§4).

### 2.3 Hierarchical prior mean (the sparsity fix)

For a cell with little data, `m` is a precision-weighted blend, most specific first:

```
m = w_st·m(source_type × claim_type) + w_s·m(source overall) + w_g·m(global)
    v1 weights: 0.5 / 0.3 / 0.2
    if the source has < 10 total resolved observations, drop the m_s term and renormalise (0.7 / 0.3)
```

With κ₀ = 8, roughly **10–20 resolved observations dominate the prior** and the cell becomes genuinely data-driven. Below that, the cell degrades gracefully to "what we know about sources like this, on claims like this" — which is an honest answer — instead of to a fabricated decimal.

*(v2: replace the fixed blend with a fitted partial-pooling model, `logit(p) = μ + a_source + b_domain + c_claimtype + interaction`, shrunk toward zero. The v1 blend is its hand-set approximation and produces the same behaviour where it matters.)*

### 2.4 Domain transferability — answered per domain, not globally

The brief asks: if a source is reliable for defence, can it be trusted for energy, technology, economics, geopolitics, manufacturing? **The model answers this structurally rather than by assertion**: transfer happens *only* through the shrinkage term, and its strength is exactly `w_s = 0.3` (source-overall) fading to zero as the target cell accumulates its own data.

Consequences, which are the correct ones:
- A source with a strong `AEROSPACE × PROC` record starts `SPACE × PROC` slightly above global (adjacent domain, same claim type, procurement skill genuinely transfers).
- The same source starts `AEROSPACE × CAP` barely above global, because **claim-type transfer is the weaker axis** — being right about who won a contract says almost nothing about being right about detection ranges.
- Reliability never transfers as a global multiplier. There is no `source.reliability` scalar anywhere in the system to abuse.

### 2.5 The resolution-rate cap (the honesty valve)

Ground truth is unevenly available, and **adversaries lie preferentially where lies cannot be falsified**. If `CAP` claims mostly never resolve and `PROC` claims always do, pooling produces "accuracy on the checkable subset" silently applied to the uncheckable subset — precisely backwards.

```
if resolution_rate(cell) < 0.10:
    L = min(L, m + 0.05)        # cannot earn much above prior in a cell that never resolves
```

Unresolved claims are **censored, not missing-at-random**. This cap is what stops the system from laundering procurement credibility into capability claims.

### 2.6 Ground-truth resolvers

| Resolver | Lag | Coverage |
|---|---|---|
| Contract databases (SAM.gov, TED, DSCA notifications) | days–months | `PROC` — excellent |
| Official confirmation / ministry statement | weeks–months | `EVENT`, `DEPLOY` — politically filtered |
| Imagery / geolocated OSINT | days–weeks | `DEPLOY`, `EVENT` |
| Authoritative correction or retraction | months | Any — strong negative label |
| Contract completion / delivery | years | `PROC`, `OOB` — definitive |
| Analyst adjudication panel | weeks | Any — expensive; budget ~10–50 claims/week |
| Declassification | decades | Operationally useless |

Weak signals (later contradiction by higher-L sources, internal inconsistency) feed a **separate `consistency` counter** and are never silently merged into the accuracy posterior.

### 2.7 Novelty weighting — do not reward stenography

A source that only reprints press releases accumulates a perfect record while adding zero information. Credit is therefore weighted by novelty at publication time:

```
ω = 1.00  if first-mover (no independent report of the claim in the prior 48h)
ω = 0.25  if echo (a prior independent cluster already asserted it)
α += ω·1  (correct)      β += ω·1  (wrong)
```

This is also an adversarial defence (§5): reputation can only be farmed by publishing *novel claims that later verify*, i.e. by burning real collection capability.

### 2.8 Asymmetric penalties

| Event | Update |
|---|---|
| Correct, novel | `α += 1` |
| Correct, echo | `α += 0.25` |
| Wrong (honest error — plausible sourcing, later corrected) | `β += 1` |
| **Confirmed fabrication** (invented artifacts, non-existent officials, forged documents) | `β += 60`, set `adversarial_flag`, trigger backward cascade (FR-11) |

Recovery from a confirmed fabrication must take longer than the reputation farm took to build. 60 pseudo-observations is that price.

---

## 3. Independence is not a profile field

`independence: 0.7` on a source record is a **category error**. Independence is pairwise and per-claim: A and B are independent today and both ran the same wire yesterday.

The full model lives in `01-architecture.md` §7 (S3): pairwise ρ table → single-linkage clustering at ρ ≥ 0.6 → one vote per cluster, weight = max member → `N_eff = N/(1+(N−1)ρ̄)`.

What belongs on the **source record** instead is the raw relational material, as three edge tables:

```
source_edge(from, to, kind ∈ {OWNS, SYNDICATES_TO, CITES, SHARES_BYLINE}, weight, observed_at)
```

Three edge tables — not a graph database, not an entity-resolution engine over world facts, not a fact store. The only thing a "knowledge graph" earns here is provenance edges; anything beyond that is building a second product inside the first.

---

## 4. Cold start

A new source appears mid-flow (G0 auto-creates it). Its prior comes from **source type + observable, hard-to-fake practices** — never from a default of trust or a default of exclusion.

**Base priors by source type** (κ₀ = 8; illustrative, to be re-anchored empirically once the ledger has data — and note the deliberate split, e.g. a government press office is strong on its own events and *weak on capability claims about its own systems*):

| Source type | `m` for `EVENT`/`PROC` | `m` for `CAP` |
|---|---|---|
| Government press office (own state) | 0.90 | 0.55 |
| Prime contractor (own product) | 0.85 | 0.45 |
| Specialist trade press | 0.84 | 0.72 |
| Wire service | 0.86 | 0.60 |
| General news | 0.72 | 0.45 |
| Think tank | 0.75 | 0.68 |
| Named OSINT analyst | 0.70 | 0.55 |
| Anonymous account / forum | 0.50 | 0.40 |
| Vendor marketing | 0.80 | 0.30 |

**Practice adjusters** (bounded, ±0.05 total on `m`): links to primary documents · named authors with verifiable history · publishes corrections · registration/domain age · distinguishes reporting from speculation.

**Both failure directions are handled automatically by the LCB:**

- *Over-trust prevented:* a new anonymous source with `m=0.5, κ₀=8` → Beta(4,4) → **L = 0.29**. Its λ contribution is ≈0.6 log-odds. It cannot swing anything.
- *Lock-out prevented:* 10 consecutive verified novel claims → Beta(14,4) → **L ≈ 0.65**. Real trust, earned in weeks.

**Two-track handling.** `collection_value` (a new source's claims are *leads*, and trigger verification tasking on day one) is separate from `evidentiary_weight` (which must be earned). A newcomer is useful immediately without being trusted immediately. *Rejected: a fixed newcomer quarantine period — it discards lead value and creates a gameable clock.*

---

## 5. Adversarial resistance

**Attack A — the 18-month reputation farm.** A state actor builds a source, accumulates a clean record, then publishes one critical falsehood. Under a naive scalar model this works perfectly: score → 0.95, poisoned claim inherits 0.95.

Four structural defences, all already in the arithmetic:

1. **Single-cluster cap `Λmax = 2.0`** (G5). No source at any reputation crosses ACCEPT alone. Maximum damage per identity is bounded by construction, not by policy.
2. **No cross-claim-type transfer.** The farm necessarily consists of cheap, *resolvable* claims (`PROC`, `EVENT`) — those are the only ones that can build a record. Under the (domain × claim-type) cut plus the §2.5 resolution cap, that record buys almost nothing on a high-stakes `CAP` or `ATTRIB` claim. **The 18 months purchase nothing where it matters.** That is the "expensive" property the design needs.
3. **Novelty weighting.** Reputation is only farmable by publishing novel claims that later verify — i.e. by expending real intelligence. Correct price.
4. **Artifact weighting over identity weighting** (S4 `a_c`). Evidence that is cheap to check and expensive to fake outranks reputation.

**Attack B — N sockpuppets that appear independent.** Defence is entirely S3: correlated content, timing lags, no independent primary artifact, longitudinal co-movement → pairwise ρ high → `N_eff ≈ 1`. Puppets face an economic bind: to build reputation under novelty weighting they must publish novel verifying claims, but copying each other yields ρ ≈ 1 and no novelty credit.

**Structural requirement, stated once:** corroboration counts *independent evidence chains terminating in primary artifacts* — never outlet count. Every metric in the system obeys this or it is wrong.

---

## 6. Source Profile — the actual record

```jsonc
{
  "source_id": "src:janes",
  "identity": {
    "display_name": "Janes",
    "type": "specialist_trade_press",
    "publisher": "org:janes-group",
    "domains_registered": ["janes.com", "..."],   // mirror/syndication resolution
    "first_seen": "2019-02-11",
    "channel_verification": {"method":"dns+tls+known_registry","verified":true,"checked_at":"..."}
  },

  // (a) COVERAGE — what they write about. Derived from corpus classification.
  //     NEVER used as trust. Selects which competence cells apply.
  "coverage": {"AEROSPACE":0.91,"NAVAL":0.62,"DEFENCE_LAND":0.55,
               "PROCUREMENT":0.88,"C4ISR_CYBER":0.40,"SPACE":0.31,"GEOPOLITICS":0.22,
               "computed_at":"2026-08-01","corpus_n":14203},

  // (b) COMPETENCE — sparse Beta posteriors. The only thing that may weight evidence.
  "competence": {
    "AEROSPACE|PROC": {"alpha":141.2,"beta":9.8,"n_resolved":142,"n_asserted":510,
                       "resolution_rate":0.28,"mean":0.935,"L":0.90,"prior_m":0.84},
    "AEROSPACE|CAP":  {"alpha":9.4,"beta":4.6,"n_resolved":6,"n_asserted":388,
                       "resolution_rate":0.015,"mean":0.671,"L":0.44,"prior_m":0.72,
                       "capped_by_resolution_rate":true},
    "NAVAL|EVENT":    {"alpha":6.7,"beta":1.3,"n_resolved":0,"resolution_rate":null,
                       "mean":0.838,"L":0.55,"prior_m":0.84,"note":"prior only"}
  },

  // (c) BEHAVIOURAL — observable practice, feeds priors and directness, not trust directly
  "practice": {"cites_primary_documents":0.71,"named_bylines":true,
               "publishes_corrections":true,"hedging_discipline":0.66,
               "median_lag_to_wire_h":-4.2},   // negative = frequently first

  // (d) RELATIONAL — the independence substrate (edges live in source_edge)
  "relations": {"owner":"org:janes-group","wire_subscriptions":["reuters"],
                "known_syndication_partners":["..."]},

  // (e) OPERATIONAL — derived view only, never an input to math
  "tier_view": {"tier":"T1","since":"2025-11-02","basis":"L(AEROSPACE|PROC)=0.90"},
  "flags": {"adversarial":false,"under_review":false,"new_source":false}
}
```

**Note the honesty in the example:** Janes is `L = 0.90` on aerospace procurement (142 resolved observations) and `L = 0.44` on aerospace capability claims (6 resolved, 1.5% resolution rate, capped). That is the distinction the brief asked for — "excellent for defence procurement, weak for radar technical specifications" — and it falls out of the model rather than being asserted.

---

## 7. Source vectorisation — what is actually vectorised

The brief asks whether to use structured metadata, embeddings, domain vectors, knowledge graphs, reliability matrices, historical performance, or a hybrid. Verdict per candidate:

| Candidate | Verdict | Role |
|---|---|---|
| Structured metadata | **Core** | Identity, type, relations. The registry. |
| Domain vector | **Keep, re-purposed** | Coverage only. Never trust. |
| Embeddings — of the source's *corpus*, to infer coverage | **Yes** | This is topic classification; it populates the coverage vector cheaply and is the honest use. |
| Embeddings — of the source as a *trust* representation | **Reject outright** | There is no training signal under which a dense vector of a source's own prose encodes reliability. Nearest neighbours cluster by topic and **style — and style is exactly what a sockpuppet copies at zero cost.** A trust embedding is a similarity attack surface: write like Janes, get scored like Janes. |
| Knowledge graph | **Narrow yes** | Exactly three edge tables (owns / syndicates / cites+bylines), because independence is a graph property. Nothing more. |
| Reliability matrix | **Yes, as posteriors** | The (domain × claim-type) Beta grid. Not point scores. |
| Historical performance | **Yes — it is the ground truth of the whole model** | The outcome ledger. Everything else is prior. |

**The pipeline the brief asked to define:**

```
Source
  → Profile          registry identity + relations + practice
  → Coverage vector  corpus classification            → which cells apply
  → Claim context    (domain, claim_type) from S1/G1  → cell selection
  → Competence cell  Beta(α,β) posterior              → L = LCB₁₀
  → Directness d, artifact a  (from S4, per claim, not per source)
  → λ_c contribution → belief P (G5)
  → outcome ledger   → posterior update → back to competence cell
```

Note that the source contributes **one number, `L`,** to the belief math. Everything else about a source is either routing, prior construction, or independence structure. Resisting the urge to let more of the profile into the arithmetic is what keeps the system auditable.

### Profile evolution

| Trigger | Effect | Latency |
|---|---|---|
| Outcome resolved | `α`/`β` update, novelty-weighted | Immediate |
| Fabrication confirmed | `β += 60`, adversarial flag, **backward cascade** over the derivation DAG | Immediate + async recompute |
| Corpus drift (source starts covering a new domain) | Coverage recomputed monthly; new cells open at hierarchical prior | Monthly |
| Ownership / syndication change | Edge tables updated; **all open provenance clusters recomputed** | On change |
| Prior re-anchoring | Source-type base priors re-fitted from the population's resolved record | Quarterly, versioned, triggers belief replay (NFR-4) |

---

## 8. T1 / T2 / T3 — verdict

**Against tiers:** they destroy information (0.89 vs 0.91 becomes a categorical cliff), invite boundary-gaming and committee fights, and — the fatal one — leak into math where they do not belong ("it's T1, skip corroboration"), which is exactly the hole Attack A drives through.

**For tiers:** routing decisions genuinely *are* discrete (auto-ingest / analyst queue / collect-only), humans and SOPs need three words rather than a posterior, and SLAs attach to categories.

**Verdict: tiers survive as a derived, read-only view. They are never an input to any calculation.**

| | Continuous posteriors | Tier view |
|---|---|---|
| Job | All math: weighting, aggregation, thresholds | Routing, SLA, human communication, dashboards |
| Derived from | Outcome ledger | `L` of the source's **strongest cell**, with hysteresis |
| May be an input to belief? | Yes — it *is* the input | **Never** |

```
promote to T1 when L ≥ 0.90        demote from T1 when L < 0.87
promote to T2 when L ≥ 0.70        demote from T2 when L < 0.67
otherwise T3
```

Hysteresis (±0.03) stops sources flapping across a boundary and stops the flap from generating churn in routing and analyst attention.

**Tier and claim confidence are separate dimensions, and must stay separate.** A T3 source with a primary artifact and two independent corroborators produces a high-confidence claim. A T1 source alone produces `P = 0.80` — provisional. Any design where tier and confidence are the same axis re-admits every failure this document exists to prevent.

---

## 9. Minimum viable v1 — no ML, no training data

Everything above ships as counting, hashing and closed-form Beta arithmetic:

- Source registry: type, ownership, syndication, bylines. Three edge tables. No embeddings, no graph DB.
- Two taxonomies: 7 domains × 7 claim types.
- Per-cell counters + hand-set source-type priors (one table, κ₀ = 8). Score = LCB₁₀. Sparse cells fall back to the hierarchical blend.
- Provenance pass per claim: MinHash, citation extraction, timestamp ordering, ownership lookup → rule ρ table → clusters.
- Resolution ledger: contract databases + official confirmations + a capped analyst adjudication queue. Every score renders with `n` and `resolution_rate`.
- Decision rule: high-impact claims need ≥ 2 independent chains, one touching a primary artifact, regardless of any source's score.
- Tier view derived from `L` with hysteresis, display and routing only.

**Deferred to v2:** fitted partial-pooling interaction terms · learned pairwise ρ from longitudinal co-movement · coverage classifier replacing hand-tagged domains · campaign detection beyond the synchrony heuristic.
