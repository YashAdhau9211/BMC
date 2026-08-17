# DFS — 06 · Gate Cascade G3 → G2 → G1

**v2 — revised after two adversarial Fable reviews.** Changes from v1 are marked **[R]** with the finding that forced them. `07-gate-review-findings.md` holds the unabridged critique.

**Standing honesty note.** Every threshold below (`S ≥ 0.35`, `R ≥ 0.55`, `P ≥ 0.85`, `ρ ≥ 0.6`) was set by tuning constants until five hand-picked examples landed where I wanted them. That is not calibration, it is circular reasoning with arithmetic on top. These are **seed values pending the calibration harvest** (§9). Treat them as placeholders with the right shape, not as measured.

---

## 1. The organising principle

**Gate number = the bar it sets. Tier number = how deep the claim got.**

```
                 volume        cost/item     question asked
  ┌──────────────────────────────────────────────────────────────────┐
  │ G3  ADMISSION      100%      cheap      is this a real, relevant,
  │                                          attributable claim at all?
  ├──────────────────────────────────────────────────────────────────┤
  │ G2  SUBSTANTIATION  ~40%     medium     is there something behind
  │                                          this other than an assertion?
  ├──────────────────────────────────────────────────────────────────┤
  │ ── ACTIVE VERIFICATION ──    async      go and fetch the record   [R]
  ├──────────────────────────────────────────────────────────────────┤
  │ G1  VERIFICATION    ~10%     expensive  does independent evidence
  │                                          converge, with no open conflict?
  └──────────────────────────────────────────────────────────────────┘

  passed G3 only        → T3   admitted, unverified, INERT
  passed G3 + G2        → T2   substantiated, one evidence chain
  passed G3 + G2 + G1   → T1   verified — enters the main knowledge system
```

**[R] The single biggest structural change in v2:** the cascade as designed only *weighs what arrives*, so its yield is capped by what the world volunteers. For a large class of defence claims the confirming primary record **already exists in a public structured database** — the claim is downstream reporting of it. Going and fetching it manufactures the second independent cluster G1 is waiting for. Active verification (§5) is now a first-class stage, not an afterthought.

Three properties retained from v1: the expensive check runs on volume that earned it; tier is an output, not a label; failure is a stop, not a delete.

---

## 2. G3 — ADMISSION

**Question:** is this a well-formed, attributable, non-duplicate, defence-relevant claim?
**Bias:** high recall. G3 removes noise, not falsehood.

### Hard checks

| # | Check | Reject reason |
|---|---|---|
| 1 | Schema valid, parseable, required fields present | `MALFORMED` |
| 2 | Source resolvable to a registry entry (auto-create with cold-start prior if new) | `UNATTRIBUTED` |
| 3 | `published_at` present and distinct from `ingested_at` | `NO_PROVENANCE_TIME` |
| 4 | Not a duplicate — content hash **and** claim fingerprint unseen | `DUPLICATE` → link, increment observation |
| 5 | Not a known spill fingerprint | `SPILL_HOLD` — legal hold, human only |

### Relevance score — **[R] the `max()` was a bypass**

v1 used `R = max(entity_hit, 0.6·clf + 0.4·centroid)`. The OR-branch meant **entity-stuffing was a total bypass**: anything mentioning "F-35" was admitted regardless of topic. And it discarded novelty at the only discarding gate — a first-surfacing classified codename has no registry entry (`entity_hit = 0`), unfamiliar vocabulary (`clf ≈ 0.5`), no prior corpus (`centroid ≈ 0.3`) → `R = 0.42` → **dropped forever**, with a 1% QA sample as the only recovery. The system was tuned to lose exactly the items with the highest information value.

```
R = min(1.0,  0.6·classifier + 0.4·centroid_similarity + 0.35·entity_hit)

  R ≥ 0.55            → ADMIT as T3
  0.35 ≤ R < 0.55     → QUARANTINE BAND: triage queue, candidate entity
                        auto-created, not discarded                        [R]
  R < 0.35            → REJECT (1% sampled for QA)
  dual_use = true     → never rejected, both domains recorded
```

Entity mention now *contributes* rather than *overrides*: an irrelevant document name-dropping a system scores 0.47 and lands in triage instead of the KB. The novel-codename case scores 0.42 and is recovered instead of destroyed.

### The self-feeding loop — **[R] v1's guard only bound honest actors**

v1 fired `SELF_DERIVED` when `dfs_fact_id ∈ origin_lineage`. **The laundering path erases lineage by construction:** DFS exports a fact → the loop paraphrases it → an external outlet picks it up ("according to reports…") → it returns with a clean external `source_id`, no fact ID, re-rounded numbers, no citation. It evades every clustering feature — no near-duplicate text, no citation link, different numeric fingerprint after re-rounding, no shared wire/byline/owner — lands as a fresh independent cluster, and **raises `k` and `P` on the very fact it derived from.** The `generation ≤ 2` bound was measured on *recorded* lineage, so it constrained only actors honest enough to preserve it. Vacuous. **Deleted.**

Four replacements, in order of value:

| # | Mechanism | Detail |
|---|---|---|
| 1 | **DFS's own export corpus joins every clustering run** | The most important near-duplicate comparison — incoming item against our own published output — never happened in v1. Cluster at ρ ≥ 0.6 against exports → inherit `SELF_DERIVED`. Cheapest fix in the review. |
| 2 | **Temporal precedence** | Record a semantic fingerprint and timestamp at export. Corroboration arriving *after* our export is discounted unless it carries evidence demonstrably predating it (document dates, imagery capture times). Independent discovery can prove pre-export provenance; laundered echo cannot. |
| 3 | **Novel-content delta** | A source that agrees but adds zero bits beyond our export is worth ~0 as corroboration *regardless of lineage*. Weight corroboration by new information, not by stance agreement. |
| 4 | **Precision canaries** | We control our export formatting. A "new independent" source reproducing DFS's exact rounding signature — where the primary used different precision — is fingerprinted derivation. |

Loop items keep their own `source_id` namespace: the loop is scored as a source like any other, not trusted for being internal.

### Output

| Outcome | Result |
|---|---|
| ADMIT | **T3.** Stored, visible, marked unverified, **inert** — cannot corroborate anything. |
| TRIAGE | Quarantine band. Human or scheduled re-scoring. Not discarded. |
| REJECT | Dropped, 1% sampled. Only discarding stage in the system. |

---

## 3. G2 — SUBSTANTIATION

**Question:** is there anything behind this claim other than someone saying it?

Source and evidence remain **one gate** — they are the same question, and splitting them lets a good source with no evidence through one door and unattributable evidence through the other.

### Hard checks

| # | Check | Fail |
|---|---|---|
| 1 | Source channel verified — an "official press release" must actually be from that channel | `UNVERIFIED_CHANNEL` |
| 2 | Evidence exists and is not the claim restated | `CIRCULAR` |
| 3 | Entity resolved ≥ 0.60 | `UNRESOLVED_ENTITY` → curator queue |
| 4 | Units normalised, no unresolved conversion artifact | `UNIT_AMBIGUOUS` → **review, never auto-fail** [R] |
| 5 | Source not flagged adversarial | `SOURCE_QUARANTINED` |
| 6 | **[R]** Hard-physics envelope not violated at z > 3 (§6) | `IMPLAUSIBLE` → stays T3, annotated with the violated envelope, analyst queue |
| 7 | **[R]** Source does not silently contradict its own earlier figures in this fact group | demote-only, lowers effective `L`; a *declared* revision is neutral |

### **[R] Evidence-class routing — the deepest fix in this revision**

v1 scored everything as `S = L · d · a`. The multiplicative form conflates two independent questions — *is this speaker calibrated?* and *is this artifact authentic?* — and gets the domain exactly backwards:

| Item | v1 arithmetic | v1 verdict |
|---|---|---|
| **Genuine leaked primary document, unknown source** (cold-start `L ≈ 0.20`) | 0.20 × 1.0 × 1.3 = **0.26** | **FAIL** |
| Trade press restating a press release | 0.80 × 0.70 × 1.0 = **0.56** | pass |
| Forged document via a compromised high-`L` channel | 0.90 × 1.0 × 1.3 = **1.17** | pass |

The highest-value item class in defence OSINT failed while its echo passed, and a forgery through a reputable channel outscored a genuine document through a new one. The `a` multiplier capped at 1.3 was a 30% nod to what should be a separate evaluation path.

**Two paths, chosen by evidence class:**

```
PATH A — ASSERTION (no verifiable primary artifact)
  S = L · d · m  ≥ 0.35
    L ∈ [0,1]    source competence, LCB of the Beta posterior for THIS
                 (domain × claim-type) cell
    d ∈ [.35,1]  directness: primary doc 1.0 · official 0.9 · trade press 0.7
                 · general news 0.5 · social/forum 0.35
    m ∈ [.7,1.3] incentive alignment (§4)

PATH B — ARTIFACT (verifiable primary artifact attached)
  Scored on AUTHENTICABILITY, not on the courier.
    checks: internal consistency · metadata/provenance chain · cross-checkable
            details against the KB · format and register conformity
            · does the document's own arithmetic close
    L enters ONLY as a forgery-prior modifier, never as a multiplier
    PASS if authenticity ≥ threshold, REGARDLESS of L
```

Path B is what recovers the leaked document. Path A keeps the bar on bare assertions intact — which is why the fix is **routing, not switching to an additive form**: additive would let a high-`L` source pass on zero evidence, which multiplication correctly prevents.

**Also note the knife-edge v1 left in Path A:** a maximally-trusted forum account's bare assertion scored `1.0 × 0.35 × 1.0 = 0.35` — exactly at threshold, where ±0.01 of estimation noise flips the outcome. And the published "OSINT analyst at 0.36 just passes" anchor was a build spec for adversaries. Thresholds now carry secret jitter (§9).

### Fetch actions at G2 — **[R] cheap, high-yield**

| Trigger | Action | Effect |
|---|---|---|
| Claim quotes a document (`a = 1.15`) not in the evidence store | Resolve and fetch it; verify the quote appears | Quote found → upgrade to Path B. **Quote not found → fabricated citation**, one of the highest-precision liar signals available: heavy `β` penalty on the source's posterior |
| Cross-language claim | Retrieve the original-language source, re-extract the quantitative fields | Mistranslation and unit slippage ("up to" silently dropped, "ordered" vs "optioned") are a top generator of false-but-sourced defence claims. Mismatch demotes and annotates |

### Provenance clustering — **[R] tightened**

Single-linkage at ρ ≥ 0.6 over near-duplicate text, citation links, identical numeric fingerprint *including rounding*, shared wire credit, byline, owner. Plus:

- **Common-origin presumption [R]** — same claim type, same values, same 72h window ⇒ presumed one origin, rebuttable by evidence of separate collection. Single-linkage on surface features alone left LLM-rewritten variants across fresh outlets as "independent"; fake cluster diversity is now cheap to manufacture.
- **Unresolvable ultimate origin [R]** — clusters whose root cannot be established get their combined λ capped as if they were one cluster.
- Weight = best member, never the sum. One cluster = one vote.

### Output

| Outcome | Result |
|---|---|
| PASS | **T2** — one evidence chain, with `incentive_class` and `evidence_class` attached. |
| FAIL | Stays **T3** with a typed reason code. Re-evaluated when new evidence enters the fact group. |

---

## 4. Incentive class — **[R] new**

Competence measures whether a source is *usually right*, not whether it has reason to lie about *this* claim. A vendor is maximally competent about its own product and maximally motivated to overstate it. v1 caught the vendor case only by accident, via a low `L` — mislabelling motive as incompetence.

```
m = f(source_affiliation, claim_subject, claim_valence)      — a function of the TRIPLE,
                                                                never of source identity

  against-interest      principal harmed by the claim          m = 1.25
                        (contractor disclosing a delay in a filing, a state admitting a loss)
  neutral               no affiliation path to the subject     m = 1.00
  self-interested       announcing own/principal's capability  m = 0.80
  adversary-derogatory  affiliated with subject's adversary,   m = 0.70
                        claim lowers the subject
```

**The counting rule is worth more than the multiplier.** At G1, when any supporting cluster is self-interested or adversary-derogatory:

```
require  distinct(incentive_class over clusters) ≥ 2
```

Three organs of the same state announcing that state's new missile are **one voice**, whatever their citation graph looks like. This closes the most exploited hole in corroboration systems and costs a small ownership table plus one line in the `k` check.

**Three locks against this becoming unfalsifiable bias:** (1) `m` is computed from the triple, so the table hits every state's press office symmetrically when each talks its own book — no per-source hand scores exist anywhere in the system; (2) bounded 0.7–1.3, so `m` alone can never flip a well-evidenced outcome; (3) **audited** — Brier score of `m`-adjusted vs unadjusted `P` on claims that later ground-truth; a class multiplier not improving calibration over ~6 months shrinks toward 1.0 automatically. The term is a hypothesis with a scoreboard, not an editorial stance.

---

## 5. Active verification — **[R] new stage, between G2 and G1**

Asynchronous worker, feeding the existing re-run machinery. Not inline: retrieval latency must not block a gate.

**Trigger:** claim is T2 **and** fact-group `P ∈ [0.50, 0.85)` **and** claim type maps to an oracle **and** budget available.
**Priority:** `Pr(record exists | type, jurisdiction) × ΔP_if_found / query_cost`. The probability starts as a hand table (US contract award ≈ 0.9, third-country MoD announcement ≈ 0.2) and updates from our own hit rate. No ML.

| Claim type | Oracle |
|---|---|
| Contract award, budget line | SAM.gov / FPDS, USAspending, EU TED, UK Contracts Finder, ministry tender portals |
| Ship claim with pennant/hull number | Naval registries, IMO/MMSI, AIS archives |
| Aircraft with tail number | Civil and military registries |
| Test / launch event | NOTAM / NAVAREA archives |
| Order or backlog | Issuer's regulatory filings, annual reports |
| Treaty-limited system | Treaty data exchanges |
| Basing, construction, hull presence | Imagery **archive** first; new tasking only from a human-approved queue |

**Match logic:** structured query on awardee + date window ± value tolerance, matched within a rounding class (press rounds $486.3M to "$0.5B"). Exact field match → full independent cluster at `d = 1.0`, `a = 1.3`. Partial match (right awardee and date, value outside tolerance) → ingested *and* raises a conflict for G1 — which is itself yield, because it catches inflated-value reporting.

**Circularity guard — essential.** The fetched record must not be the ancestor of the claim's existing cluster. Check citation links and publication ordering; if the claim's cluster cites the registry entry, the fetch upgrades `d` but **does not increment `k`**. Without this the retrieval loop counterfeits independence — the exact failure the whole design exists to prevent.

**Expected yield:** press-reported contract awards, hull/tail-numbered deliveries, and NOTAM'd test events are claims whose primary record is public and machine-readable. For those, the bottleneck was never truth — it was that nobody went and looked.

**Registry negatives.** A registry hand-tagged *complete* for its domain returning no record is near-decisive: `λ = −2`. Completeness is a human-assigned, periodically reviewed per-registry attribute; most registries are not complete and get no negative power.

---

## 6. Consistency checks — **[R] new**

### Physical and engineering envelopes

A versioned library of hand-coded envelope functions, each returning `z` = distance outside the envelope in envelope-widths. Runs at G2 (extreme screen) and G1 (graded).

| Tier | Examples | Power |
|---|---|---|
| **Hard physics** | Radar range vs power-aperture (`R⁴ ∝ P·Aₑ²/σ`, bounding P by achievable array power density for the stated band and aperture) · missile range vs mass class (rocket equation, propellant fraction ≤ 0.9, Isp bounded by known chemistry) · payload–range–fuel closure | May hard-block at `z > 3` |
| **Engineering practice** | Delivery schedule vs known line rate × 1.5 surge · deployment count vs apron/shelter capacity · build duration vs class-historical keel-to-commissioning bands | Demote only |
| **Accounting** | Order count × unit-cost band vs announced programme budget, tolerance [0.5×, 1.5×] because "programme cost" ambiguously includes sustainment | Demote only |

```
z ≤ 1        no effect
1 < z ≤ 3    λ_physics = clamp(−2·(z−1), −2, 0)
z > 3        hard-physics tier → G2 blocks, stays T3, annotated, analyst queue
             other tiers      → demote only
```

**Strictly asymmetric: a plausibility check can FAIL a claim, never PASS one.** Consistency with priors is free for a fabricator — any competent liar writes plausible numbers — so passing carries ~zero information while violating carries a lot.

**Breakthrough protection**, because a wrong prior rejecting real news is the obvious failure: only hard-physics may block; "it costs too little" and "the line can't run that fast" are exactly the priors a breakthrough breaks, so they only demote. Every hard-blocked claim routes to an analyst with the specific violated envelope named. Envelopes are versioned artifacts — correcting one is an evidence event that triggers re-runs, so claims blocked by a wrong prior **auto-repromote when the prior is fixed**.

### KB consistency

Runs at G1, shares one channel `λ_KB`, clamped `[+1.0, −∞)` — positive credit weaker than a real source cluster, negative unbounded so a ledger impossibility is decisive. **`λ_KB` never counts toward `k`**, or derived evidence starts substituting for observation.

| Check | Logic |
|---|---|
| **Arithmetic closure** | Running ledgers: fleet = prior + deliveries − losses − retirements; deliveries ≤ orders; budget draw ≤ appropriation. Residual zero within rounding → **+0.5** (exact closure is not free to fake without knowing the other figures — the only safe positive prior credit in the system). Impossible residual → hard negative |
| **Timeline templates** | Per claim-type ordering and duration bands (contract → first flight → IOC; keel → launch → commissioning). Ordering violations are hard negatives; duration outliers are soft |
| **Relationship prerequisites** | Claim asserts an edge whose prerequisites are missing from the KB. **Does not demote** — absence of a KB edge is usually absence of KB coverage. Instead it emits a retrieval query into §5's queue. Consistency checking becomes a trigger generator for active verification, which is where its yield actually is |

**Poisoning guard, non-optional:** positive `λ_KB` creates a feedback loop where one false T1 fact makes consistent false claims cheaper forever. Only T1 facts whose supporting clusters span ≥ 2 incentive classes may generate positive `λ_KB`, and taint propagation (§8) must ship **before** positive `λ_KB` does.

---

## 7. G1 — VERIFICATION

### Hard checks

| # | Check | Fail |
|---|---|---|
| 1 | `k ≥ 2` independent provenance clusters — **or the attested exception below** | `SINGLE_CHAIN` |
| 2 | **[R]** `distinct(incentive_class) ≥ 2` when any cluster is self-interested or adversary-derogatory | `ONE_VOICE` |
| 3 | Comparability — variant resolved, qualifiers sufficient. **[R]** Conflict may only block *after* variant resolution passes at high confidence, or Block-I/Block-II specs merge into a phantom conflict that holds two true claims forever | `NOT_COMPARABLE` |
| 4 | Conflict classified and closed (unit → entity → qualifier → precision → temporal, in order) | `OPEN_CONFLICT` |
| 5 | Temporally valid — **[R]** conflict detection only within overlapping validity intervals, or G1 burns budget adjudicating 2023's fleet count against 2026's and blocks true state changes | `STALE` |
| 6 | No `campaign_suspected` flag | `CAMPAIGN_FLAG` |
| 7 | **[R]** At least one supporting cluster contains a primary artifact | `NO_ARTIFACT` |

### Confidence

```
λ_c   = clamp( 2.2 · L_c · d_c · a_c · m_c · s_c , −2.0, +2.0 )
ℓ     = logit(π_claim_type) + Σ λ_c + λ_physics + λ_KB + λ_exposure
P     = σ(ℓ)
```

**[R] `π` is now per claim-type, not a fixed 0.35.** One prior for everything meant "routine unit rotation" (high base rate) and "sixth-gen fighter flew five years early" (tiny base rate) needed identical evidence to reach 0.85.

**[R] `λ_exposure` — score exposure, never specificity.** Specificity is free to fabricate; a flat precision bonus is a published instruction to add fake hull numbers. So an *unchecked* specific claim scores identically to a vague one — checkable atoms affect only **priority in the retrieval queue** (§5). Each atom checked-and-survived: **+0.3** (≤ +1.0 total). Each atom checked-and-**failed**: **−1.0**, plus an outsized `β` penalty on the source. Specificity becomes pure upside for honest sources and a minefield the fabricator planted under themselves.

**[R] Vague claims cap at T2 regardless of `k`.** "Significantly extended range" corroborates with everything and means nothing; convergence on vague claims is free.

### **[R] The `k ≥ 2` exception — T1-ATTESTED**

v1's `k ≥ 2` was fatal as specified. Every contract award, budget line, order announcement and doctrine statement has exactly **one** authoritative origin: the announcing state. Nobody can independently confirm France's own budget. Under v1 the T1 layer would be structurally empty of procurement facts — the domain the system exists for — while being populated by multiply-echoed press-release derivatives.

Admit at **T1-ATTESTED** (distinct export status) only if **all five** hold:

1. **Claim class is performative/institutional** — the announcement *constitutes* the fact: contract awards, budget allocations, orders placed, official designations, force-structure decisions. These cannot be wrong in the ordinary sense, only forged. **Explicitly excluded: any world-state or capability claim** ("range is 400 km", "achieved IOC performance X") — those describe physics, the announcer has incentive to distort, and they stay under `k ≥ 2`.
2. **Source is the subject** — about actor A, from A's channel-of-record. No third party may ever invoke this, including trade press quoting the official.
3. **Channel authentication is hard** — pre-registered channel-of-record registry, out-of-band verified (domain control, document signatures, journal-of-record status).
4. **Primary artifact required** — the document, not a summary.
5. **Falsification hook mandatory** — an expected downstream observable and window (deliveries, spending lines, follow-on filings). Deadline passes unmet, or a contradicting observable appears → automatic demotion plus a channel-trust penalty. `P` capped at **0.90**: attested, never converged.

**Why this is not a laundering path:** the gate is *class-based, not score-based*. An adversary cannot phrase a capability lie as performative (condition 1 is a claim-type classifier plus review for critical items), cannot use a proxy outlet (condition 2), and must compromise an actual official channel rather than mimic one (condition 3) — and a compromised official channel defeats any verification design, so this adds no attack surface that "official primary document = strongest evidence" didn't already have. *Rejected: a higher `P` bar for `k = 1` generally — score-based exceptions admit every high-`L` single-source scoop including vendor claims and farmed personas.*

Single-provider imagery is a different problem: imagery is an artifact checkable against physics (orbital mechanics, shadow geometry, weather archives). A second competent analysis counts as a second cluster only if the analyst is infrastructurally distinct **and adds novel verification content** — corroborating the artifact, not repeating the claim.

### **[R] Asymmetric refutation**

v1 required `k_refute ≥ 2` **and** `P < 0.35` to mark REFUTED. Check the arithmetic against denial asymmetry — states rarely bother denying: one maximal refuter (−2.0) against two trade-press supporters (+1.232 each) gives `ℓ = −0.619 + 2.464 − 2.0 = −0.155` → `P = 0.46` → **HOLD**. A single authoritative refutation could not kill a claim two echoes supported, so false claims were effectively immortal at exported-provisional.

```
one authoritative refuting PRIMARY ARTIFACT  →  demote T1→T2 AND suspend export
                                                immediately, pending adjudication
provisional exports EXPIRE if P has not moved upward within the claim-type window
```

### Decision

| Condition | Outcome |
|---|---|
| `impact = critical` and not (`P ≥ 0.95` and `k ≥ 3`) | **REVIEW** — human, 4h SLA |
| Attested exception satisfied | **T1-ATTESTED**, `P ≤ 0.90` |
| `P ≥ 0.85`, `k ≥ 2`, all hard checks pass | **T1** |
| Authoritative primary-artifact refutation | **REFUTED** — export suspended |
| otherwise | **HOLD at T2** with a typed reason |

---

## 8. Tiers, export, and re-entry

### **[R] T2 was five populations wearing one label**

v1 exported all of them as `status: provisional`: (a) sole-authoritative-source awaiting impossible corroboration — probably true; (b) live unresolved conflict — contested; (c) stale/superseded — formerly true; (d) `campaign_suspected` — probably an info-op; (e) freshly substantiated, G1 pending — unknown. A consumer could not distinguish the French budget line from a suspected influence operation. Worse, **exporting (d) as provisional made DFS an amplification channel for the campaign it had just flagged** — the adversary's claim shipped with a quasi-endorsement.

```
export payload = (tier, hold_reason, P, k, evidence_class, incentive_classes, as_of)
                  hold_reason is TYPED and MANDATORY on every non-T1 export
                  campaign_suspected NEVER exports, at any tier
```

Tier stays a scalar processing-depth state — that part was sound. The diagnosis rides in the reason code. *Rejected: a full tier-vector across evidence/conflict/freshness/adversariality — consumers will not correctly interpret four axes, most cells are unreachable, and scalar depth + typed reason + `P` carries the same information with far less interface surface.*

**[R] Also fixed: T2 punned two unrelated meanings** — a pipeline status *and* "one evidence chain" (a G1 input unit). Now separate fields.

### What each tier may do

| | **T3** | **T2** | **T1** |
|---|---|---|---|
| Counts as an evidence chain | **no** | yes — one | yes |
| Can corroborate another claim | **no** | yes | yes |
| Exported | no | with typed `hold_reason` | yes |
| Can generate positive `λ_KB` | no | no | only if clusters span ≥ 2 incentive classes |

**T3 claims are inert** — held so they can be promoted, never so they can vote.

### Re-entry — **[R] rate-limited**

v1's "any new claim in a fact group re-runs G2 for all T3 and G1 for all T2" was a denial-of-service: an attacker pays G3-cheap prices to force G1-expensive re-runs, and the 100/40/10 cost model assumes a volume distribution the adversary controls.

```
re-runs trigger ONLY on items that themselves pass G2          [R]
per-fact-group re-evaluation rate limits                        [R]
T1 demotion requires human review, never automatic on a flag    [R]
```

That last one blocks **demotion poisoning** — injecting adversarial-flagged items to knock true T1 facts down.

**Promotion is still free in this structure.** A weak source is first and correct, fails Path A at G2, sits at T3. Weeks later corroboration arrives, G1 sees `k = 2`, the fact becomes T1 — and the original T3 claim is credited as **first-mover** in the outcome ledger, raising that source's `L` so it passes next time.

### **[R] Taint propagation — build before positive `λ_KB` ships**

T1 was write-monotone in v1. Every T1 fact stores its supporting cluster set; when a source is later adversarial-flagged or an atom fails a check, walk the dependency graph and re-run every fact it supported — **including facts that earned positive `λ_KB` from the now-tainted fact.** Without this, one successful plant compounds forever.

---

## 9. Calibration and anti-gaming — **[R]**

**The calibration harvest is quietly the highest-value item in this document.** A useful fraction of claims ground-truth themselves within months: the delivery happens or doesn't, the contract surfaces in FPDS, the ship appears on AIS. Harvest those outcomes as labels and recalibrate everything currently set by folklore — the 0.35 and 0.85 thresholds, the 2.2 coefficient, the ±2.0 clamp, `L` posteriors, `m` multipliers, `d` values.

**Log resolved outcomes from day one even though recalibration comes later. The data is free only if collected from the start.**

Two anti-gaming rules, because published constants are a build spec:

- **Threshold-bunching detector** — monitor score distributions for mass accumulating just above any threshold. Bunching is itself a campaign signature; feed it to the campaign flag.
- **Secret jitter** on threshold values so the bar cannot be learned exactly.

---

## 10. What to build first

| # | Item | Why first |
|---|---|---|
| 1 | **Active verification: contract/registry oracles + quoted-document fetch** (§5) | Largest T2→T1 conversion per unit cost. A few connectors to free structured databases. The fetch doubles as a fabricated-citation detector |
| 2 | **DFS export corpus in every clustering run** (§2) | One line of scope change; closes the loop-laundering hole that nothing else detects |
| 3 | **Arithmetic closure ledgers** (§6) | Pure bookkeeping over data the KB already holds. Hard negatives on an entire fabrication class, plus the only safe positive credit |
| 4 | **Incentive-class counting rule** (§4) | Small `k`-check change plus an ownership table. Closes the many-mouths hole |
| 5 | **Evidence-class routing at G2** (§3) | Recovers the highest-value item class the v1 arithmetic was discarding |
| 6 | **Typed hold reasons + campaign never exports** (§8) | Turns T2 from undifferentiated purgatory — and an info-op amplifier — into an interpretable state |
| 7 | **Time-versioned facts** (§7) | Schema work, but unblocks true state changes currently strangled by phantom conflicts |
| 8 | **Physics envelopes, hard tier first** (§6) | ~12 formulas, high-precision demotion, with the breakthrough escape hatch |
| 9 | **Taint propagation** (§8) | Insurance against compounding poison. Build *before* positive `λ_KB` ships |
| 10 | **Calibration harvest logging** (§9) | Start collecting on day one; recalibrate later |

**The unifying principle:** the v1 cascade only *weighed what arrived*, so its yield was capped by what the world volunteered. Everything above either goes and gets the missing evidence, makes lying expensive in a way being honest isn't, or stops counterfeit independence from impersonating corroboration. Tweaking `L`, `d` and `a` is third-order next to those three moves.
