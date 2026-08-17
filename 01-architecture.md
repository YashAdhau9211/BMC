# DFS — 01 · System Architecture & Gate Logic

---

## 1. Verdict on the proposed pipeline

Proposed: `Input → G1 Context → G2 Source → G3 Evidence → Trusted Output`

**Three structural faults:**

1. **Wrong primary key.** The pipeline gates *documents*; every downstream question ("is this source trustworthy *for this claim*", "do these conflict") is a *claim-level* question. One Janes article contains 15 claims of wildly different reliability — hull count (solid), engine thrust (copied from a brochure), delivery date (speculation). One verdict for all 15 is wrong for most of them.
2. **Source as a gate is self-sealing.** Hard-rejecting a low-tier source before its evidence is examined loses the scoop, loses the timeline, and — fatally — removes the outcome that would have *raised* that source's score. Low-tier sources stay low-tier forever because their correct claims are destroyed before scoring. It also hands adversaries a spec: launder through a T1-adjacent channel and skip scrutiny.
3. **Missing the highest-value stage entirely.** Most apparent conflicts in this domain are not disagreements — they are qualifier mismatches (different target RCS, different Pd, different variant). Without a normalisation/comparability stage the system manufactures conflicts and then "resolves" them into numbers no source asserted.

**Restructured as: two hard gates, four scorers, one decision point, one feedback loop.**

## 2. Stage map

```
                                    ┌─────────────────────────────────────┐
                                    │  SOURCE REGISTRY + COMPETENCE CELLS │
                                    │  (Beta posteriors, edges, aliases)  │
                                    └───────────┬─────────────────────────┘
                                                │ read                ▲ write
                                                ▼                     │
 Document ──► G0 ──► G1 ──► S1 ──► S2 ──► S3 ──► S4 ──► G5 ──► S6 ──► L
              │      │      │      │      │      │      │      │      │
   Intake ────┘      │      │      │      │      │      │      │      │
   Relevance ────────┘      │      │      │      │      │      │      │
   Claim extract+normalise ─┘      │      │      │      │      │      │
   Entity/variant resolve ─────────┘      │      │      │      │      │
   Provenance/independence cluster ───────┘      │      │      │      │
   Evidence assessment ──────────────────────────┘      │      │      │
   Belief + publication DECISION ───────────────────────┘      │      │
   KB reconciliation (bitemporal write) ───────────────────────┘      │
   Outcome ledger + reputation update ────────────────────────────────┘

 HARD GATES (may terminate flow): G0 (structural), G1 (relevance), G5 (decision)
 SCORERS  (annotate, never reject): S1 S2 S3 S4
 EVIDENCE LAYER: append-only, written at S1, never mutated
 BELIEF LAYER:   derived at G5/S6, always recomputable
```

**Why this shape.** Everything before G5 *accumulates annotation*; only G5 decides. That kills the 3^N state explosion of per-gate PASS/FAIL/REVIEW and makes the decision policy a single tunable, auditable artifact rather than four interacting ones.

---

## 3. G0 — Intake & Provenance Capture

**Hard gate. Structural validity only — no judgement.**

| | |
|---|---|
| **Input** | Document per §7 of the PRD. |
| **Checks** | 1. Required fields present. 2. `source_id` exists in registry (unknown → auto-create with cold-start prior, flag `new_source`). 3. Content hash dedup. 4. Charset/parse success. 5. Spill detection (classification markings regex, known-leak fingerprints). 6. Timestamp sanity (`published_at ≤ retrieved_at + skew`). |
| **Derived at capture** (unrecoverable later — this is why G0 exists) | MinHash/SimHash text fingerprint · numeric-token fingerprint (value **+ unit + rounding pattern** — copied numbers survive rewording) · citation extraction (hyperlinks, "according to X", wire credits) · byline set · publisher identity resolved across mirrors and syndication · dual timestamps. |
| **Score** | None. Boolean. |
| **Outcome** | `PASS` → S-relevance. `DUPLICATE` → link to existing doc, increment observation, stop. `MALFORMED` → dead-letter queue with reason (retryable). `SPILL_SUSPECTED` → legal-hold quarantine, human only, no downstream flow. |
| **Output** | `Document` row + `provenance_features` row. |

> Rationale: the echo problem is only solvable if citations, fingerprints and *both* timestamps are captured at ingest. A copy-detection heuristic bolted on in v2 cannot reconstruct a derivation graph retroactively.

---

## 4. G1 — Relevance & Scope

**Hard gate. The only stage permitted to discard content (cost control).**

| | |
|---|---|
| **Input** | Document text + title + source registry domain-coverage vector. |
| **Checks** | 1. Defence entity registry match (systems, programmes, units, primes, designators, NATO reporting names). 2. Topic classifier over the defence taxonomy. 3. Embedding similarity to per-domain centroids. 4. Dual-use registry match (semiconductors, UAS, space launch, cyber tooling, PNT, advanced materials). |
| **Logic** | `entity_hit` = 1.0 if a registry entity appears with a defence-context term within N tokens. `R = max(entity_hit, 0.6·clf + 0.4·centroid_sim)`. Dual-use match forces `R ≥ 0.5` and sets `dual_use=true` with **both** civil and military domains recorded. |
| **Score** | `R ∈ [0,1]`, plus `domains[]` with per-domain scores, plus `dual_use`, plus `ambiguity = 1 − (top_domain − second_domain)`. |
| **Thresholds** | `R ≥ 0.60` → PASS. `0.35 ≤ R < 0.60` → PASS tagged `scope: peripheral` (extracted, but claims start at a lower prior and never auto-ACCEPT). `R < 0.35` → REJECT. |
| **Exceptions** | `dual_use=true` → never REJECT. `ambiguity > 0.4` → PASS tagged `domain_ambiguous`, routed to multi-domain scoring (claim inherits the *lower* of the applicable source competence cells). |
| **QA** | 1% of REJECTs retained and sampled weekly. Rejection is a write of zero bits — it must be measured, because early filters silently bias what the KB can ever contain. |
| **Outcome** | `PASS` / `PASS_PERIPHERAL` / `REJECT(sampled)` |

> "Is this defence-related?" is not one question. It is *which* defence domains, at what strength, with what ambiguity, and is it dual-use — because the answer selects which source competence cells apply downstream.

---

## 5. S1 — Claim Extraction & Normalisation

**Scorer. The highest-value stage in the system, and the one entirely absent from the original design.**

| | |
|---|---|
| **Input** | PASSed document + domain tags. |
| **Mechanism** | LLM, **caged**: schema-constrained output, one atomic claim per assertion, every claim must carry byte-offset spans. Then a **deterministic validator** rejects any claim whose span does not exist, or whose numeric token does not string-match the source text. Unit conversion, date parsing, and designator canonicalisation are **deterministic only** — an LLM converting nmi→km wrong is a silent, plausible, unfindable corruption. |
| **Claim atom** | `(entity_ref, variant_ref, claim_type, attribute, value, unit, granularity, qualifier_vector, stance, hedging, asserted_valid_time, span, doc_id)` |
| **Qualifier vector** (measurement claims) | quantity_type · target_rcs · target_class · altitude/aspect · environment (clear/clutter/rain/ECM) · mode · statistical definition (Pd, Pfa, single-scan vs cumulative) · number_provenance (brochure / acceptance test / operational / analyst estimate / simulation) · config baseline + as-of date. |
| **The UNKNOWN rule** | Absent qualifiers are stored as `{value: null, state: "UNKNOWN"}`. **Never imputed into the assertion.** Convention-based imputations ("probably 1 m² RCS") are written to a separate `hypotheses` field, flagged, and are *excluded from the contradiction test*. |
| **Checks** | span existence · numeric string match · unit in canonical set · **conversion-artifact detector** (value ratios vs 1.852 / 1.609 / 0.3048 / 0.5399 within 1%) · per-attribute plausibility band (a fighter radar "detection range" of 12 km or 4000 km trips a flag) · hedging detection ("reportedly", "up to", "sources say") → recorded as `assertion_strength`. |
| **Score** | `extraction_confidence` (validator-derived, not LLM self-report) · `qualification_completeness = qualifiers_known / qualifiers_relevant_to_quantity_type`. |
| **Thresholds** | Span validation failure → claim **discarded** (extraction error, not information). `qualification_completeness < 0.3` on a measurement claim → claim tagged `weakly_qualified`: it may **corroborate** a fully-qualified claim, but may **never contradict** one. |
| **Outcome** | 0..N claim atoms appended to the immutable evidence layer. |

**The asymmetric comparability rule (FR-13).** A claim with unknown qualifiers can support but cannot refute. This single rule is what prevents the system from manufacturing the 200/210/230 "conflict" out of three claims that were never about the same quantity.

---

## 6. S2 — Entity & Variant Resolution

**Scorer.**

| | |
|---|---|
| **Input** | Claim atoms with raw entity strings. |
| **Mechanism** | Curated alias/designator authority file + retrieval/similarity for candidates. **The LLM may propose candidate links; it may never commit one.** A hallucinated entity merge poisons every corroboration count downstream and is close to undetectable afterwards. |
| **Checks** | Alias table (S-400 = Triumf = SA-21 Growler) · transliteration variants (Cyrillic/Arabic/Chinese romanisation) · NATO reporting name ↔ native designator · hull/tail numbers ↔ class · **variant/block discrimination** (Su-35 vs Su-35S, Block I vs II, domestic vs export config) · operator-country disambiguation. |
| **Score** | `entity_confidence`, `variant_confidence`. |
| **Thresholds** | `entity_confidence ≥ 0.9` → resolved. `0.6–0.9` → resolved with `entity_uncertain` flag; claim can corroborate but not refute. `< 0.6` → **PARKED** as `unresolved_entity` — retained in the evidence layer, excluded from all fact groups, queued for curator resolution. Variant unresolved but entity resolved → attaches to the entity with `variant: UNKNOWN`; may corroborate, never refute a variant-specific fact. |
| **Outcome** | Resolved / uncertain / parked. Parked ≠ rejected. |

> Entity resolution failure is bidirectionally fatal: one claim looks like three (missed corroboration) or two different systems merge (false corroboration). It is upstream of every count in the system.

---

## 7. S3 — Provenance & Independence Clustering

**Scorer. This is the echo killer, and it replaces "Independence" as a source-profile field.**

Independence is **pairwise and per-claim**, not a property of one source. A source-profile field `independence: 0.7` cannot express "A and B are independent today but both ran the same wire copy yesterday."

**Pairwise correlation table (v1, rule-based):**

| Signal | ρ |
|---|---|
| Same wire story / syndication credit | 0.95 |
| Near-duplicate text (MinHash Jaccard ≥ 0.8) | 0.90 |
| B explicitly cites A (link or "first reported by") | 0.85 |
| Identical numeric fingerprint — same value, unit **and rounding** | 0.80 |
| Shared byline | 0.70 |
| Shared ownership / parent | 0.60 |
| Published within 6h after, no new detail | 0.50 |
| Otherwise | 0.00 |

**Clustering:** single-linkage at `ρ ≥ 0.6`. Each cluster casts **one vote**. Cluster weight = **max** of members, never sum. Cluster root = earliest publication in the derivation DAG.

**Effective sample size (reported, and used from v2 in weighting):**

```
N_eff = N / (1 + (N−1)·ρ̄)        (equivalently 1ᵀ Σ⁻¹ 1 for the full matrix)
```

Sanity check that must hold: 40 outlets carrying one AP wire → ρ̄ ≈ 0.95 → **N_eff ≈ 1.03**. Any aggregation that does not produce ≈1 here is broken.

**Also detected here:**
- **Shared upstream dependence** — two "independent" outlets both fed by one ministry briefing are one origin. Detected via briefing-time clustering + identical quote sets.
- **Coordinated-campaign signal** — many sources, same claim, tight synchrony, *no* citation chain, no primary artifact. This raises `campaign_suspected` and **suppresses** the corroboration boost rather than granting it. Naive systems read a seeding operation as strong consensus; this is where that is stopped.

| Score | Outcome |
|---|---|
| `k` = independent cluster count, `N_eff`, `raw_source_count`, `campaign_suspected` | Annotation only — S3 never rejects. |

---

## 8. S4 — Evidence Assessment

**Scorer.**

| | |
|---|---|
| **Input** | Claim + retrieved candidate evidence (attached artifacts, cited documents, prior KB facts, corpus retrieval). |
| **Mechanism** | Retrieval for candidates; LLM in **NLI mode** (supports / refutes / neutral) with a constrained verdict and mandatory cited spans; a deterministic validator string-matches every quoted span. This is a legitimate LLM strength, kept honest by grounding. |
| **Checks** | Does the cited passage actually contain the claimed support? · Is the "evidence" the claim restated (circular)? · Is a primary artifact attached and independently checkable (filing, imagery, award notice, contract document)? · Does the claim survive physical-prior sanity (e.g. asserted range vs plausible power-aperture)? |
| **Scores** | `stance ∈ [−1,+1]` (signed, × NLI confidence) · `directness d` (primary document 1.0 · official statement 0.9 · trade press 0.7 · general news 0.5 · social/forum 0.35) · `artifact multiplier a` (verifiable primary artifact 1.3 · quoted document 1.15 · bare assertion 1.0). |
| **Outcome** | Annotation. Circular evidence is marked and contributes zero. |

> **Weigh cost-to-fake, not just source identity.** A claim carrying a verifiable primary artifact outranks a bare assertion from a 0.95-reputation source. This moves weight from reputation (farmable over 18 months) to artifacts (expensive to fabricate) and is a primary adversarial defence.

---

## 9. G5 — Belief Computation & Publication Decision

**The single decision point. Deterministic. No LLM anywhere in this stage (FR-10, NFR-1).**

### 9.1 Confidence formula

Work in log-odds. For each independent provenance cluster `c`:

```
λ_c = clamp( κ · L_c · d_c · a_c · s_c , −Λmax, +Λmax )

  κ     = 2.2      evidence gain constant (calibrated, config)
  Λmax  = 2.0      SINGLE-CLUSTER CAP  ← this is FR-4, enforced in the arithmetic
  L_c   ∈ [0,1]    lower-confidence-bound of the best member's competence posterior
                   for THIS (domain × claim-type) cell  — see 02-source-model.md
  d_c   ∈ [.35,1]  provenance directness (S4)
  a_c   ∈ [1,1.3]  artifact multiplier (S4)
  s_c   ∈ [−1,+1]  stance × NLI confidence (S4)

ℓ  = logit(π_ct) + Σ_c λ_c            π_ct = claim-type base rate, default 0.35
P  = σ(ℓ)
```

**Worked checks (these are the design's calibration anchors):**

| Situation | ℓ | P | Verdict |
|---|---|---|---|
| One trade-press cluster, L=0.80, d=0.70, a=1.0, s=+1 | −0.62 + 1.23 = 0.61 | **0.65** | Single source ≠ accepted. |
| Two such independent clusters | −0.62 + 2.46 = 1.84 | **0.86** | Meets bar with k=2. |
| One official primary document, L=0.90, d=1.0, a=1.3 (λ=2.57 → **capped 2.0**) | −0.62 + 2.00 = 1.38 | **0.80** | Even a perfect single source lands below ACCEPT. FR-4 holds arithmetically. |
| 40 outlets on one wire (collapses to k=1, L=0.8, d=0.5) | −0.62 + 0.88 = 0.26 | **0.56** | The echo problem, priced correctly. |

### 9.2 Decision matrix

Evaluated in order; first match wins.

| # | Condition | Outcome | Meaning |
|---|---|---|---|
| 1 | `spill_suspected` or `campaign_suspected` or source `adversarial_flag` | **QUARANTINE** | Stored, flagged, excluded from belief, human-only. |
| 2 | `impact = critical` AND NOT (`P ≥ 0.95` AND `k ≥ 3`) | **REVIEW** | Human, SLA 4h. Confidence cannot buy past this. |
| 3 | `impact = sensitive` AND (`P < 0.85` OR `k < 2`) | **REVIEW** | SLA 24h. |
| 4 | Refuting clusters with `k_refute ≥ 2` and `P < 0.35` | **REFUTED** | Stored as refuted, published as negative evidence. |
| 5 | `P ≥ 0.85` AND `k ≥ 2` AND no unresolved contradiction | **ACCEPT** | Publishes to KB. |
| 6 | `P ≥ 0.85` AND `k < 2` | **PROVISIONAL** | Status `single_source`. Visible, not trusted. |
| 7 | `0.55 ≤ P < 0.85` | **PROVISIONAL** | |
| 8 | `P < 0.55` | **HOLD** | Quarantine-retained. Re-evaluated on every new related claim. **Not deleted** — this is the scoop's holding pen. |

**Nothing here deletes.** HOLD is the mechanism that makes the scoop recoverable: when independent confirmation arrives later, S6 matches it against held claims, promotes them, and — critically — retro-credits the *origin* source's reputation in that cell (see S7). First-and-right must be worth more reputation than fortieth-and-right, and the S3 derivation DAG is exactly what makes "who was first" computable.

**Reason codes** are emitted with every decision (`SINGLE_PRIMARY_CHAIN`, `ECHO_COLLAPSED`, `DISSENT_PRESENT`, `WEAKLY_QUALIFIED`, `CAMPAIGN_SUSPECTED`, `IMPACT_FORCED_REVIEW`, …) and the human-readable `reason` string is templated from them — never LLM prose.

---

## 10. S6 — KB Reconciliation

**Bitemporal write. Detail in `03-conflict-and-belief.md`.**

Classifies the incoming claim against what is already stored: `novel` / `corroborating` / `contradicting` / `superseding` / `stale`. This gate catches the failure mode that a well-run pipeline still produces: confidently ingesting a **stale truth**. Without it the KB silently accumulates contradictions and every downstream consumer re-does the reconciliation, badly, N times.

## 11. S7 — Outcome Ledger & Reputation Update

**The feedback loop the original design has no edge for.**

Resolvers (contract databases, official confirmation, imagery, retraction, delivery, analyst adjudication) write outcomes; outcomes update the (source × domain × claim-type) posteriors; **retractions and source compromise cascade backwards** through the derivation DAG, recomputing every fact that depended on the affected claims (FR-11). Detail in `02-source-model.md`.

---

## 12. LLM vs deterministic boundary

**Rule: LLMs propose, deterministic code disposes. No LLM output reaches the KB without a deterministic validator, and no confidence number is ever produced by an LLM.**

| Stage | Mechanism | Why |
|---|---|---|
| G0 intake, dedup, fingerprints | Deterministic | No judgement involved. |
| G1 relevance | Cheap classifier + embedding similarity | Needs throughput and a stable, tunable threshold; LLM is overkill and drifts. |
| S1 claim extraction | **LLM, caged** + span validator | Only technology that handles the variety. The cage is what makes it safe. |
| S1 normalisation (units, dates, designators) | **Deterministic only** | An LLM unit conversion error is silent, plausible, and unfindable. |
| S2 entity resolution | Alias tables + retrieval; LLM may *propose*, never commit | Hallucinated merges poison every corroboration count. |
| S3 independence/echo | Deterministic (graph + hashing) | Must be auditable and stable across runs. |
| S4 evidence assessment | LLM in NLI mode, span-verified | Legitimate strength, grounded. |
| **G5 belief & confidence** | **Deterministic formula — the single most dangerous place for an LLM in this system** | Two runs returning 0.7 and 0.9 on identical inputs makes every downstream decision arbitrary and destroys audit and regression testing (NFR-1). |
| S6 reconciliation | Deterministic on normalised values; LLM only *flags* semantic-contradiction candidates for humans | Value comparison is code. Nuance goes to people. |
| S7 reputation | **Deterministic** from the outcome ledger | Asking an LLM "is this source trustworthy" returns training-data fame, not measured accuracy. |
| Contradiction adjudication, critical accepts, disinfo calls | **Human** | Accountability requires a name attached. |
