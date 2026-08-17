# DFS — 05 · Architectural Decision Log (Fable Advisory)

Fable was consulted as a **design adviser only** — three independent adversarial reviews of (a) gate architecture, (b) source modelling, (c) conflict resolution, each briefed to attack rather than validate the hypotheses in the original brief.

**Fable is not a runtime component of DFS.** It has no place in the pipeline: belief computation must be deterministic and reproducible (NFR-1), and no LLM may produce a confidence number (FR-10). Its role is design-time challenge, and the standing advisory points for future consultation are listed in §3.

Format per the brief: **Decision → Alternatives → Evidence → Fable recommendation → Final decision → Reason.**

---

## 1. Major decisions

### D-01 · Claim extraction becomes its own stage (S1), upstream of everything
- **Alternatives:** (a) document-level gating as originally proposed; (b) extraction after factuality assessment.
- **Evidence:** one trade article contains ~15 claims of wildly different reliability — hull count (solid), engine thrust (copied from a brochure), delivery date (speculation). Every downstream question in the brief is claim-scoped.
- **Fable:** "Non-negotiable. The four-box diagram operates on documents, but per-claim trust, corroboration counting and conflict reconciliation are all impossible at document granularity — you'd be building the whole system on the wrong primary key."
- **Final:** Adopted. S1 with span grounding, byte offsets, and a deterministic validator.
- **Reason:** Wrong primary key is not a fixable defect later; it invalidates every count in the system.

### D-02 · Source verification demoted from GATE to WEIGHT
- **Alternatives:** (a) G2 as a hard pass/fail gate (original); (b) gate with a quarantine appeal channel; (c) weight in aggregation.
- **Evidence:** defence OSINT scoops disproportionately originate from low-tier sources. A hard gate loses the claim, loses the timeline, and destroys the outcome that would have raised the source's score — the gate is **self-sealing**: low-tier sources can never improve because their correct claims are discarded before scoring.
- **Fable:** "The worst decision in this design… a hard gate hands adversaries a spec: launder disinfo through a T1-adjacent channel and it skips scrutiny; the tier system becomes the attack surface." On the appeal channel: "the exception path becomes the main path and you maintain two pipelines."
- **Final:** (c). Source trust enters only as `L` in the λ contribution at G5. The only hard early reject is relevance (G1), which samples its rejects.
- **Reason:** Fixes the scoop problem, removes a self-sealing feedback loop, and shrinks the adversarial attack surface — at the cost of storing claims we do not publish, which is cheap.

### D-03 · Per-gate PASS/FAIL/REVIEW replaced by scores + one decision point
- **Alternatives:** (a) three-way outcome per gate (original); (b) scores throughout, single decision policy at G5.
- **Evidence:** 3^N states is unmanageable at N=4 and fatal at the realistic N=7; and the states are not meaningful anyway — gates are not independent, so "REVIEW at G1 but PASS at G3" has no defined semantics.
- **Fable:** rejection is "a write of zero bits — you can't retro-score what you didn't keep"; a static early gate is "learnable by an adversary".
- **Final:** (b). Scorers annotate; G5 decides; HOLD replaces reject.
- **Reason:** One tunable, auditable decision policy instead of four interacting ones. Early rejection also silently biases the KB toward whatever the cheapest filter likes.

### D-04 · Add a KB reconciliation stage after factuality (S6)
- **Alternatives:** (a) three gates, publish straight to KB; (b) add reconciliation.
- **Evidence:** a claim can be individually well-evidenced and still contradict or be superseded by what is stored.
- **Fable:** "Without it the KB silently accumulates contradictions and downstream consumers do the reconciliation ad hoc, badly, N times… this gate is also where you catch the failure mode of a well-run pipeline: confidently ingesting a *stale* truth."
- **Final:** (b). Adopted, with bitemporal storage.
- **Reason:** Three gates were insufficient; the brief explicitly asked whether they were.

### D-05 · Source domain vector re-purposed: COVERAGE, not competence
- **Alternatives:** (a) domain vector as trust (original); (b) drop it; (c) split coverage from competence.
- **Evidence:** hand-assigned 0.88 vs 0.92 is not reproducible by humans (±0.1 inter-rater at best); learned values need ~50–100 resolved claims/cell, which most cells will never reach.
- **Fable:** "A spreadsheet with false precision… coverage ≠ competence. A source can emit 10,000 aerospace articles and be wrong in all of them. Use it only to decide *which cells exist*, never to set *cell values*."
- **Final:** (c). Coverage vector from corpus classification selects cells; competence posteriors weight evidence.
- **Reason:** Keeps the useful part of the original idea and removes the dishonest part.

### D-06 · Point scores replaced by Beta-Binomial posteriors, scored by lower confidence bound
- **Alternatives:** (a) scalar per domain; (b) scalar + a separate confidence field; (c) Beta posterior per cell, `L = LCB₁₀`.
- **Evidence:** 3/3 → Beta(4,1), mean 0.80, **LCB₁₀ 0.56**. 285/300 → Beta(286,16), mean 0.947, **LCB₁₀ 0.93**. The point score ranks the lucky newcomer *above* the seasoned source; the LCB does not.
- **Fable:** "Closed form, incremental, auditable… and it solves cold start for free."
- **Final:** (c). κ₀ = 8, hierarchical prior blend, LCB₁₀ is the only number reaching belief math.
- **Reason:** Sample size is information, and the alternative discards it. Also removes the need for training data in v1.

### D-07 · The cut is (domain × claim-type), with hierarchical pooling
- **Alternatives:** (a) domain only; (b) claim-type only; (c) domain × claim-type with shrinkage.
- **Evidence:** the brief's own example — excellent on procurement, weak on radar specs — cannot be expressed by (a). 7×7 = 49 mostly-empty cells is the cost.
- **Fable:** "The naive matrix dies of sparsity… sparse cells should degrade gracefully to 'what we know about sources like this on claims like this' instead of to garbage."
- **Final:** (c). Fixed blend (0.5 source-type×claim-type / 0.3 source overall / 0.2 global) in v1; fitted partial pooling in v2.
- **Reason:** This is the exact distinction the brief required, and pooling makes it affordable.

### D-08 · Trust embeddings rejected outright
- **Alternatives:** (a) embed sources as trust representations; (b) embed corpora for coverage only; (c) no embeddings.
- **Evidence:** there is no training signal under which a dense vector of a source's own prose encodes reliability; embedding neighbours cluster by topic and style.
- **Fable:** "Style is precisely what a state-actor sockpuppet copies perfectly at zero cost. A trust embedding is a similarity attack surface: write like Janes, get scored like Janes."
- **Final:** (b). Embeddings for coverage inference only.
- **Reason:** A trust representation that an adversary can imitate for free is worse than no trust representation.

### D-09 · Knowledge graph reduced to three provenance edge tables
- **Alternatives:** (a) full defence-facts knowledge graph; (b) provenance edges only; (c) none.
- **Evidence:** independence is a graph property, so *some* graph is needed. Nothing else in the design requires graph traversal.
- **Fable:** "A full KG of defence facts is building a second product inside the first, and its ground-truth problem is the same unsolved problem" as source accuracy.
- **Final:** (b). `OWNS / SYNDICATES_TO / CITES / SHARES_BYLINE` in Postgres. No graph database.
- **Reason:** Buy exactly the capability the independence model needs and nothing else.

### D-10 · Independence removed from the source profile; modelled pairwise per claim
- **Alternatives:** (a) `independence: 0.7` as a profile dimension (original); (b) pairwise ρ + clustering.
- **Evidence:** two outlets are independent today and both ran the same wire yesterday. A per-source scalar cannot represent this.
- **Fable:** "A category error… independence of A depends on who else reported the claim today." And the arithmetic: 40 outlets on one AP wire → ρ̄ ≈ 0.95 → **N_eff ≈ 1.03**. "Any aggregation that doesn't produce that number is broken."
- **Final:** (b). Rule-based ρ table → single-linkage clustering at ρ ≥ 0.6 → one vote per cluster, weight = max member. `N_eff = N/(1+(N−1)ρ̄)` reported alongside.
- **Reason:** The echo problem is the single highest-frequency failure in this domain, and only a per-claim structure solves it.

### D-11 · Single-cluster influence cap (`Λmax = 2.0`)
- **Alternatives:** (a) unbounded reputation weighting; (b) hard cap on any single cluster's log-odds contribution.
- **Evidence:** an 18-month reputation farm under unbounded weighting works perfectly — score reaches 0.95 and the poisoned claim inherits it. With the cap, a perfect single source reaches `P = 0.80`: below ACCEPT.
- **Fable:** "This bounds the maximum damage of any single farmed identity **by construction**" — and combined with no cross-claim-type transfer, "the 18 months purchase nothing where it matters."
- **Final:** (b). `Λmax = 2.0`, plus the 60% single-cluster weight cap in conflict resolution.
- **Reason:** Adversarial resistance must be arithmetic, not policy. Policy is negotiable under pressure; a clamp is not.

### D-12 · Reputation credit weighted by novelty
- **Alternatives:** (a) all correct claims credited equally; (b) novelty-weighted (first-mover 1.0, echo 0.25).
- **Evidence:** a source that only reprints press releases accumulates a perfect record while adding zero information.
- **Fable:** "Without this, the system trains the ecosystem toward stenography." And: sockpuppets "must publish novel claims that later verify — i.e. burn real intelligence — because copying each other yields ρ ≈ 1 and no novelty credit."
- **Final:** (b).
- **Reason:** Correct incentive for honest sources, and it prices reputation-farming in actual collection capability.

### D-13 · Resolution-rate cap on competence scores
- **Alternatives:** (a) pool all resolved outcomes; (b) cap `L` near prior in cells with `resolution_rate < 0.10`.
- **Evidence:** procurement claims resolve in weeks; capability claims mostly never resolve. Pooling produces "accuracy on the checkable subset" applied silently to the uncheckable subset.
- **Fable:** "Adversaries lie preferentially where lies can't be falsified… unresolved claims are *censored*, not missing-at-random."
- **Final:** (b).
- **Reason:** Prevents laundering procurement credibility into capability claims — which is precisely where deliberate deception concentrates.

### D-14 · T1/T2/T3 retained as a derived view only
- **Alternatives:** (a) tiers as the primary model; (b) abolish tiers; (c) tiers derived from `L` with hysteresis, display and routing only.
- **Evidence:** tiers quantise (0.89 vs 0.91 becomes a cliff), invite gaming, and leak into math ("it's T1, skip corroboration"). But routing genuinely is discrete and humans need three words, not a posterior.
- **Fable:** "Tiers must never appear as an input to any calculation… every aggregation formula then quantises its inputs and the cliff effects propagate into conclusions."
- **Final:** (c). Promote T1 at `L ≥ 0.90`, demote at `< 0.87`; T2 at 0.70/0.67. **Never an input to belief.**
- **Reason:** Keeps the operational value, removes the mathematical harm. Directly answers the brief: **source tier and claim confidence are separate dimensions and must stay separate.**

### D-15 · Conflict resolution reframed as a comparability problem
- **Alternatives:** (a) resolve 200/210/230 as conflicting values; (b) establish comparability first, resolve only the residue.
- **Evidence:** `R ∝ σ^(1/4)` — 200 km at 1 m² and ~283 km at 4 m² are *the same claim*. A 15% spread is inside what a Pd definition change alone produces.
- **Fable:** "The majority of apparent numeric conflicts in this domain are qualifier mismatches, not disagreements… the proposed design spends its machinery on genuine disagreement and will therefore mostly be mis-applying it."
- **Final:** (b). Ordered taxonomy: unit artifact → entity/variant → qualifier mismatch → precision → temporal → genuine → disinformation.
- **Reason:** This is the largest single accuracy win available, and it moves budget from a resolver nobody needed to an extractor everybody does.

### D-16 · Never average; consensus-with-dissent (medoid, not mean)
- **Alternatives:** (a) weighted mean; (b) inverse-variance pooling; (c) most-attested asserted value + recorded dissent.
- **Evidence:** 213.3 km appears in no document and corresponds to no test condition. One nmi contamination (200 nmi = 370 km) silently wrecks any mean. Inverse-variance needs per-source variances that are unreported, unestimable under copying, and **reward precision laundering** — a copy re-reporting "≈200" as "204" gets up-weighted.
- **Fable:** "A number no source asserts, no document contains, and no engineer would defend."
- **Final:** (c). Stored values are always asserted values. Intervals are a rendering, never the record.
- **Reason:** Auditability. "We believe 200 km per the acceptance test report; Janes cites 230, a brochure figure" is defensible. "213.3" is not.

### D-17 · Evidence layer and belief layer separated
- **Alternatives:** (a) single mutable current-value store; (b) append-only evidence + derived, recomputable belief.
- **Evidence:** three routine events require recomputation — the algorithm improves, a new source flips an old dispute, a source is exposed as a disinformation channel. All three are impossible if resolution destroyed its inputs.
- **Fable:** "'Resolved' means classified, with a current belief computed — never evidence discarded." Costs: ~2–5× storage, materialised views, fiddly versioning semantics, and consumer friction over `disputed` — "that last cost is the point of the system."
- **Final:** (b), with bitemporal fact versions.
- **Reason:** A defence KB whose values cannot be traced to evidence at a point in time is unusable for anything that matters.

### D-18 · LLM confined to extraction, NLI, and candidate proposal
- **Alternatives:** (a) LLM for source judgement and confidence scoring; (b) strict boundary with deterministic disposal.
- **Evidence:** two runs returning 0.7 and 0.9 on identical inputs makes every downstream decision arbitrary and destroys audit, regression testing and replay (NFR-1, NFR-4).
- **Fable:** "Confidence aggregation is the most dangerous place for an LLM in the whole system." On unit conversion: "silent, plausible, unfindable corruption." On source reputation: "asking an LLM 'is this source trustworthy' returns training-data fame, not measured accuracy."
- **Final:** (b). **LLMs propose, deterministic code disposes.** No LLM output reaches the KB without passing a validator; no confidence number is ever LLM-produced.
- **Reason:** Non-determinism anywhere in belief computation makes the entire evaluation and rollback strategy impossible.

### D-19 · Reputation (M2) sequenced after echo detection (M1)
- **Alternatives:** (a) build reputation early to gate ingest; (b) echo detection first, static type priors until M2.
- **Evidence:** reputation computed over uncollapsed copies trains on amplification, not accuracy.
- **Fable:** implied by the echo analysis — corroboration must be computed over the derivation graph before any count means anything.
- **Final:** (b).
- **Reason:** A reputation model poisoned from the start is far more expensive to unwind than one built late.

### D-20 · Fable is design-time only, not a runtime component
- **Alternatives:** (a) Fable as a runtime adjudicator for hard cases; (b) advisory only.
- **Evidence:** D-18 forbids LLM-produced confidence; runtime adjudication would reintroduce non-determinism precisely at the highest-stakes decisions.
- **Final:** (b). Hard cases route to **humans**, whose decisions are captured in `review_task` and feed reputation.
- **Reason:** Accountability requires a name attached. An LLM cannot be accountable for a critical claim reaching a targeting-adjacent product.

---

## 2. Fable's ranked priorities across the three reviews

All three independent reviews converged on the same three items, which is the strongest signal in this exercise:

| Rank | Change | Reviews agreeing |
|---|---|---|
| 1 | **Span-grounded claim extraction + qualifier normalisation upstream of everything** — the wrong primary key invalidates the rest | Gates ✓ Conflict ✓ (Source ✓ implicitly, via the claim-type cut) |
| 2 | **Independence as a per-claim provenance structure, with `N_eff` in all aggregation** — otherwise every metric measures copying | Gates ✓ Source ✓ Conflict ✓ |
| 3 | **Append-only evidence + derived belief + never average + never delete** — makes recomputation, retraction and improvement possible at all | Gates ✓ Source ✓ Conflict ✓ |

---

## 3. Standing advisory points — where to consult Fable again

| Trigger | What to challenge |
|---|---|
| Before M3 threshold-setting | The G5 decision matrix boundaries and `κ`, `Λmax` values against the first real calibration data |
| When cell coverage passes 30% | Whether to switch from the fixed hierarchical blend to a fitted partial-pooling model, and what would falsify that switch |
| First adversarial-suite failure | Whether the failure is a threshold problem or a structural one — Fable's value is distinguishing these |
| Before adding any new gate | Whether it belongs as a gate, a scorer, or a reason code; default answer should be "reason code" |
| Any proposal to let an LLM touch scoring | Re-run D-18. This decision should be expensive to reverse |
| Before multilingual expansion | Entity-resolution failure modes across transliteration, which will silently break corroboration counts in both directions |
| Quarterly | Missing failure cases: "what's in the pipeline now that nothing in the adversarial suite tests?" |

**How to brief Fable:** give it the current design as a hypothesis and instruct it to attack, not validate; ask explicitly for the alternative it rejected and why; ask for the top three changes ranked. Reviews briefed to "review" produce validation; reviews briefed to "refute" produce the material in this log.
