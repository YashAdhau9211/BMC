# DFS — 03 · Comparability, Conflict Resolution & the Belief Layer

---

## 1. The premise correction

The brief's canonical example:

> Radar X detection range — Source A: 200 km · Source B: 210 km · Source C: 230 km

**These are usually not three answers to one question. They are correct answers to three different questions.**

Detection range is not a property of a radar. It is a property of a `(radar, target, geometry, environment, mode, statistical definition)` tuple. The radar range equation makes this quantitative: `R ∝ σ^(1/4)` where σ is target RCS. So 200 km against 1 m² and 283 km against 4 m² are **the same claim**. A 200 vs 230 spread is 15% — comfortably inside what a change of Pd definition (0.5 vs 0.9), Swerling model, or altitude line-of-sight limit produces.

**Therefore: the majority of apparent conflicts in this domain are qualifier mismatches, not disagreements.** Conflict resolution is the small residual problem. Qualifier extraction (S1) is the main problem, and it is where the engineering and ontology budget belongs.

A system that treats these three as a conflict and "resolves" them destroys information and emits a number no engineer would defend.

---

## 2. The comparability contract

Two claims enter the same fact group only if they are *about the same thing*. Grouping key:

```
(entity_id, variant_id, attribute, quantity_type, qualifier_signature, validity_epoch)
```

`qualifier_signature` is computed over the qualifier dimensions **known to move that quantity type** (a per-attribute sensitivity table — for `detection_range`: RCS, Pd, mode, environment, altitude; for `max_speed`: altitude, configuration, weapons load).

### The asymmetric comparability rule

| Claim state | May corroborate? | May contradict? |
|---|---|---|
| Fully qualified, signature match | Yes | Yes |
| Fully qualified, signature differs on a sensitivity dimension | No — **separate fact** | No |
| Weakly qualified (`completeness < 0.3`) | Yes, at reduced weight | **Never** |
| Entity resolved, variant UNKNOWN | Yes, against entity-level fact | Never, against variant-level fact |
| Entity unresolved | No | No — parked |

Three options existed for missing qualifiers; two are wrong:

- ❌ **Treat unknown as wildcard and declare conflict** → everything is flagged, the queue drowns, analysts stop reading it.
- ❌ **Silently impute** ("probably 1 m², that's conventional") → fabricated evidence that hardens into fact.
- ✅ **Asymmetric rule + flagged hypotheses.** Imputations live in a separate `hypotheses` field, are labelled with their basis ("assumed 1 m² RCS, convention-based, low confidence"), and are **excluded from the contradiction test**.

Consequence: many claim groups terminate in `not_comparable — insufficient qualification`. **That is a feature and a required output state (FR-13).** The original design has no such state, which is why it would have been forced to invent answers.

---

## 3. Conflict taxonomy — classify before resolving

Discriminating tests run **in this order**, cheapest and most specific first. Most conflicts never reach the bottom.

| # | Type | Discriminating test | Resolution |
|---|---|---|---|
| 1 | **Unit artifact** | Value ratio ≈ a known constant within 1%: 1.852 (nmi→km), 1.609 (mi→km), 0.3048 (ft→m), 0.4536, 1.0936 | Normalise, re-run from step 1. **Never a conflict.** |
| 2 | **Entity/variant confusion** | Different variant, block, export config, operator, or era. Check designator strings, dates, operator country | Split into separate entity facts. No conflict existed. |
| 3 | **Qualifier mismatch** | Qualifier signatures differ on a sensitivity dimension; **or** values are mutually consistent under `R ∝ σ^(1/4)` rescaling (or the attribute's own sensitivity law) | Store as separate qualified facts. No conflict existed. |
| 4 | **Precision / rounding** | `\|a−b\| ≤ (g_a+g_b)/2` where granularity `g` is inferred from trailing zeros and hedges ("approximately", "up to") | Not a conflict. Store the most precise **credibly-sourced asserted** value; others count as corroboration. |
| 5 | **Temporal** | Claims cluster by report date; the later cluster is internally consistent; a causal event exists (upgrade contract, block change, new variant, re-baselined test) | **Version the fact.** Both true, disjoint validity intervals. |
| 6 | **Genuine disagreement** | Comparable qualifiers, same epoch, spread beyond rounding, no unit/entity/temporal explanation | Weighted belief with dissent preserved (§4). |
| 7 | **Disinformation** | Single-origin provenance cluster · motivated source (operator's state media, adversary) · violates physical priors (claimed range implies implausible power-aperture) · timed to an event (arms fair, crisis, budget cycle) | **Quarantine**: retained in the evidence layer, flagged, **excluded from belief**. Never averaged in. |

Note the asymmetry: types 1–4 are decided by cheap deterministic tests and **dissolve** the apparent conflict. Only what survives all four reaches 5–7, where judgement and weighting live. The original design spends all its machinery on type 6 and would therefore mis-apply it to types 1–4 — which are the common cases.

---

## 4. The resolution algorithm

### 4.1 Pipeline

```
1. EXTRACT     claim → (entity, variant, quantity_type, value, unit, granularity,
                        qualifier_vector with explicit UNKNOWNs, provenance,
                        report_date, asserted_valid_time)          [S1]
2. NORMALISE   units, with the unit-artifact ratio check run BEFORE conversion is trusted [S1]
3. RESOLVE     entity + variant; unresolvable variant → parked, not conflicted            [S2]
4. GROUP       by (entity, variant, attribute, quantity_type, qualifier_signature, epoch)
5. CLUSTER     provenance within the group → one vote per cluster                         [S3]
6. CLASSIFY    taxonomy §3, in order
7. BELIEVE     per type, §4.2–4.4
8. STORE       evidence append-only; belief as a derived, versioned view                  [S6]
```

### 4.2 Point vs interval vs disputed — the decision rule

Independent clusters `c₁…c_k` with weights `w_i` (from `L_c · d_c · a_c`), `W = Σw_i`:

| Condition | Status | Stored value |
|---|---|---|
| `k = 1` | `single_source` | Point value; uncertainty = reporting granularity. **Never** "confirmed". |
| `k ≥ 2`, all within combined granularity | `corroborated` | Most precise asserted value from the most credible cluster. |
| Values disagree, top value's supporting weight ≥ ⅔·W | `consensus_with_dissent` | Consensus value **plus explicit dissent list** (value, sources, weight). |
| Values disagree, no ⅔ majority | `disputed` | The **value set**. Not an interval. |
| Only quarantined evidence exists | `quarantined_only` | No belief value published. |
| Grouping failed on qualifiers | `not_comparable` | No belief value published. |

**Single-cluster weight cap:** no cluster may account for more than **60%** of `W` on the strength of its tier alone. Consequence: `k` independent lower-`L` clusters can force `disputed` even when they cannot win. **High reputation buys the tie-break, not immunity from dispute.** This is the guardrail for "one wrong official spec sheet outvotes twelve correct observers".

### 4.3 Averaging: never. Here is what instead.

`(200+210+230)/3 = 213.3 km` is a number **no source asserts, no document contains, and no engineer would defend** — it corresponds to no test condition and no configuration. It is also fragile: one nmi-contaminated claim (200 nmi = 370 km) silently drags the mean into garbage.

Averaging is defensible only for repeated independent measurements of one physical quantity under one condition — which is exactly what step 4 has established these are *not*.

**Instead: consensus-with-dissent, medoid not mean.** The stored value is always an **asserted** value — the best-attested one — with dissenting asserted values recorded alongside with their provenance. This is auditable ("we believe 200 km per the acceptance test report; Janes cites 230 km, a brochure figure"), degrades gracefully, and never invents data.

**Inverse-variance / precision weighting: rejected.** It requires each source's error variance, which is (i) never reported, (ii) not estimable from repetition because sources copy, and (iii) actively misleading — a copied number inherits the original's typographic precision, so precision-weighting **rewards precision laundering**: a secondary source re-reporting "≈200 km" as "204 km" gets *up*-weighted. Inverse-variance is the right tool for independent instruments with known noise. Open-source defence reporting is neither.

**Intervals: rendering, never record.** Storing `[200, 230]` as *the fact* implies the truth is uniformly somewhere inside and invites midpoint-reading — averaging by another name. Intervals may be *displayed* for a disputed value set; they are not the record.

### 4.4 Outliers and the scoop

**An outlier is never deleted from the evidence layer.** The only question is whether it enters belief.

Exclude from belief when: it matches a unit-conversion ratio (type 1) · its source is quarantined (type 7) · it is temporally stale against a versioned fact (type 5).

**Preserve and flag otherwise — because the outlier is sometimes the truth.** The outlier is likely correct when all three hold:

1. It is **more specific** than the consensus (exact figure, named test, cited document vs round brochure numbers).
2. Its provenance is **upstream** of the consensus (primary document vs secondary reporting).
3. The consensus **collapses to one provenance cluster** (everyone copied the same old press release).

A single leaked acceptance-test report showing 160 km should beat nine outlets recycling a 200 km brochure figure. Under cluster-voting with directness weighting, it does: that is **1 direct cluster vs 1 indirect cluster**, not 1 vs 9. Majority-rules kills scoops; provenance-directness saves them.

### 4.5 Tie-break order (weights within 15%)

1. **Provenance directness** — primary document > official statement > trade press > aggregator.
2. **Recency** — only for facts about *current* configuration, and only within the same validity epoch.
3. **Qualifier specificity** — a fully-qualified claim beats a vague one.
4. **Internal consistency** with the entity's other stored facts (range-equation sanity against known power/aperture).
5. Still tied → **`disputed`. No forced winner.** A forced winner at 51/49 is a lie about confidence.

Never tie-break by averaging. Never tie-break by raw source count — that re-admits copy cascades through the back door.

---

## 5. Non-numeric conflicts — four different problems

The canonical example is numeric; most defence claims are not. **None of these admit averaging or weighted combination.** The type system dispatches on claim shape *before* any resolver runs.

### 5.1 Counts — "ordered 24 aircraft"

Discrete and exact, and usually reconcilable by **arithmetic decomposition rather than adjudication**: 24 = 18 firm + 6 options; 36 = original 24 + follow-on 12; "50" = cumulative across tranches.

**Logic:** model orders structurally (firm / option / LOI / tranche / cumulative-vs-incremental). The conflict test is *does a decomposition exist that makes all claims true?* If yes → store the structured breakdown, no conflict. If no → the latest authoritative procurement document wins outright. Averaging 24 and 36 into 30 aircraft is not merely wrong, it is absurd. **No partial credit for counts.**

### 5.2 Existence / location — "deployed to base Z"

A time-bound state, not a static fact — deployments end.

- Evidence tiers are **physical-first**: commercial imagery > official announcement > press report > social media.
- **Absence of reporting is not evidence of absence.** A negation ("no longer at Z") requires explicit positive evidence (imagery of empty revetments, a withdrawal statement) and even then **closes a validity interval** rather than deleting the deployment fact.

### 5.3 Categorical — "contract awarded to vendor W"

A canonical ground-truth document class exists (award notice, filings), so this is mostly a **stage-model** problem, not a credibility problem. Apparent conflicts are usually different lifecycle stages (RFP shortlist ≠ downselect ≠ award ≠ signature) or different roles (prime vs subcontractor vs local partner).

**Logic:** model the procurement lifecycle explicitly; map each claim to a stage and role; conflicts *within* the same stage+role are rare and go to document-tier tie-break. **No weighting arithmetic at all.**

### 5.4 State change / negation — "programme cancelled"

A state machine with transition evidence. Cancellation claims are asymmetric: cheap to assert, frequently reversed (funding restored, programme renamed and continued).

- Transition evidence must be proportional to irreversibility. "Cancelled" from a budget document outranks ten press claims.
- A cancellation **never erases prior state** — it closes the `active` interval.
- Watch the **rename-evasion** case: "Programme P cancelled" + "Programme Q started", where Q is P re-badged. This is an entity-resolution problem (S2), not a state conflict, and must be routed there.

---

## 6. Time: bitemporal or the KB is broken on arrival

A single mutable current value is **disqualifying**. Records are bitemporal: **valid time** (when true of the world) and **record time** (when we learned it), append-only supersession, never overwrite.

What overwriting destroys:

- **As-of queries** — "what was the assessed range in 2022?" is a core analyst question and the entire basis of trend analysis.
- **Retraction processing** — you cannot re-derive what you believed if you deleted why you believed it.
- **The change/error distinction** — the discriminating evidence *is* the history you deleted.
- **Audit** — a defence KB whose values cannot be traced to evidence at a point in time is unusable for anything that matters.

### "The fact changed" vs "the earlier report was wrong"

| Signal | Fact changed | Report was wrong |
|---|---|---|
| Causal event in the record (upgrade contract, block change, MLU) | Present | Absent |
| Old sources retract | No — old value stays true *of the old baseline* | Often yes, or the source goes silent |
| Qualifiers | Different as-of / config baseline | Same baseline |
| Transition date | Clusters near the causal event | No clustering |
| Provenance of the old value | Multiple clusters | Usually one cluster |

**The decisive test: what date does the new claim assert its value *for*?**
- Asserts the present, after an upgrade → **change.** Close the old interval, open a new one. Both remain true.
- Asserts the past ("the range was always 210; the 230 figure was a misread brochure") → **correction.** Mark the old assertion `superseded_as_erroneous` at record time; valid-time history shows it was never true.

Ambiguous → stored as `revised — change_vs_correction_undetermined` and queued for a human. **v1 does not guess this.**

---

## 7. Evidence layer vs belief layer

**The KB never collapses a disputed claim into a bare value.**

| | Evidence layer | Belief layer |
|---|---|---|
| Mutability | **Append-only, immutable** | Derived, recomputable, versioned |
| Contents | Claims: source, text span, extracted value, qualifiers with explicit unknowns, provenance chain, report date | Best current estimate, status, confidence, validity interval, supporting/dissenting pointers |
| Deletion | Never — quarantined disinformation stays, flagged | N/A — versions supersede |
| Produced by | S1 | G5 + S6 |

**Why the separation is non-negotiable here.** All three of these are routine in this domain, and all three are impossible if resolution destroyed its inputs:

1. The resolution algorithm improves — old beliefs must be recomputed under the new one.
2. A new source arrives and flips an old dispute.
3. A source is later exposed as a disinformation channel and **everything it touched must be recomputed** (FR-11).

So **"resolved" means: classified, with a current belief computed.** It never means evidence discarded.

**What the consumer reads:** the belief view — value + status + confidence + as-of, with dissent inline when status demands it, and an evidence link for drill-down. **Consumers must be forced to see status.** An API that returns `range_km: 200` without `status: disputed` will be treated as ground truth downstream — and that is how a contested brochure figure ends up in a targeting-adjacent product.

**Costs, stated honestly:** ~2–5× storage (cheap — these are text claims, not imagery) · every read passes through a materialised view (recomputed per-fact on new evidence; not a hot path) · belief-layer versioning on top of evidence bitemporality is genuinely fiddly and is the real engineering cost · and consumers will push back on `disputed`, because it is an answer people do not want. **That last cost is the point of the system.**

---

## 8. Failure modes and guardrails

| # | Silent failure | Guardrail |
|---|---|---|
| 1 | **Fake consensus from copying** — nine outlets recycle one release, engine sees n=9 | Provenance clustering, one vote per cluster (S3); copy detection via identical value + identical unusual precision + phrase overlap; corroboration reported in **clusters**, and `raw_source_count` shown beside `n_eff` so the gap is visible |
| 2 | **One wrong high-tier source outvoting many correct low-tier ones** | 60% single-cluster weight cap (§4.2) → `k` lower-tier clusters can force `disputed`. Authority buys the tie-break, not immunity |
| 3 | **Unit errors masquerading as disagreement** — 200 nmi stored as 200, "conflicts" with 370 km | Conversion-ratio detector runs **before** conflict classification (taxonomy step 1); per-attribute plausibility bands catch decimal and unit errors together |
| 4 | **Entity confusion masquerading as disagreement** — Su-35 vs Su-35S, Block I vs II | Variant-aware resolution as a hard step; unresolved variant → parked as `not_comparable`, excluded from conflict groups; qualifier signature includes config baseline |
| 5 | **Stale values winning on authority** — a 1998 manual outvotes 2024 testing of the upgraded set | Authority weight applies **only within a validity epoch**; the temporal test (taxonomy step 5) runs before any weighting, so pre- and post-upgrade claims never compete |
| 6 | **Precision laundering** — "≈200 km" copied and re-reported as "204 km" gains weight | Precision granularity is **capped by provenance** — a claim can never be more precise than its upstream origin; suspicious precision (secondary source, exact figure, no cited document) is down-flagged |
| 7 | **Silent qualifier imputation** — the extractor "helpfully" assumes 1 m² and it hardens into fact | Unknowns stored as unknowns; imputations are separate flagged hypotheses excluded from the contradiction test (§2) |
| 8 | **Disputed collapsing to a value at the API boundary** | No value-only read path exists (FR-12); `status` is mandatory in every response schema |

---

## 9. Alternatives rejected

| Rejected | Why |
|---|---|
| Inverse-variance / Bayesian precision pooling | Requires unknowable per-source variances; destroyed by correlation; rewards precision laundering |
| Majority vote / raw source counting | Measures media amplification, not truth; killed by copy cascades; kills scoops |
| Pure trust-rank (global source authority ordering) | Authority is per-claim-type and per-epoch; institutionalises stale official figures; single-point failure on one wrong authoritative document |
| Off-the-shelf truth-discovery algorithms (TruthFinder / CRH family), unmodified | Their two core assumptions — source independence and a single true value — are precisely the two this domain violates. Usable only *after* normalisation and clustering, i.e. after the hard work is done |
| Single mutable current-value store | Fails every point in §6. Disqualifying |
| Interval-as-the-record | Invites midpoint reading — averaging by another name |

---

## 10. Minimum viable v1

- Append-only claims table: entity, variant, quantity type, value, unit, precision granularity, ~6 nullable qualifier fields with **explicit unknown states**, source, provenance link, report date, asserted-as-of date.
- Deterministic normaliser: unit conversion with conversion-artifact detection; per-attribute plausibility bands.
- Copy detection by fingerprint (identical value + precision + text overlap) → provenance clusters, one vote each.
- Rule-based belief view, no learning: taxonomy tests §3 in order, then the §4.2 decision rule with the 60% cap.
- Simple validity intervals (`valid_from` / `valid_to` + `record_time`) instead of full bitemporal joins.
- `status` mandatory in every read path.

**Deferred:** probabilistic disinformation modelling beyond source flags · full bitemporal query semantics · automated change-vs-correction classification (v1 marks these `undetermined` for a human) · learned qualifier sensitivity tables (v1 hand-writes them per attribute).

That v1 already avoids every failure mode in §8 except the subtlest — which is more than the originally proposed design achieves with considerably more machinery.
