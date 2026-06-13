# Delivery Plan & Sprint Breakdown — Multimodal OSINT Chatbot POC

| Field | Value |
|---|---|
| Document | Project Delivery Plan (PM-owned) |
| Version | 1.0 — June 2026 |
| Drives | POC PRD v1.1 (open-source / Ollama) + Incremental POC Plan |
| Duration | 13 sprints × 2 weeks = 26 weeks (Sprint 0 + Sprints 1–12 across 6 phase gates) |
| Cadence | 2-week sprints; gates align to phase boundaries |
| Methodology | Scrum-ban: scrum ceremonies + a continuously-pulled eval/post-mortem lane |

---

## 1. How this plan is structured

The PRD defines *what* and *why*; this plan defines *who, when, in what order, and how we know we're on track*. Three rules govern everything:

1. **Gates, not calendars, define done.** A phase closes when its gate (G1–G6 from the PRD) passes on the held-out golden set — not when the sprint ends. If a gate slips, we de-scope `Should`/`Could` items, never the gate.
2. **The eval harness is built first and runs forever.** Every sprint from S1 reports golden-set deltas. No feature is "done" until its eval moves (or is shown not to regress).
3. **Vertical slices over horizontal layers.** Each sprint ships something an analyst can use end-to-end, even if narrow. We never spend a sprint building "just the embedding layer."

---

## 2. Team, roles & RACI

| Role | Person (placeholder) | Allocation | Primary ownership |
|---|---|---|---|
| **PM** (me) | — | Full | Backlog, gates, stakeholder comms, risk, scope control |
| **Backend / Infra Eng** | ENG-1 | Full | Ingestion pipeline, services, infra, deployment, Ollama/runtimes |
| **ML / Retrieval Eng** | ENG-2 | Full | Retrieval, rerank, graph, agents, verification, eval harness |
| **Full-stack Eng** | ENG-3 | 0.5–1.0 | UI, citation drill-down, trace/entity/graph views |
| **Analyst SME** | SME | ~2 days/sprint | Golden-set authoring, gate review, credibility taxonomy, domain truth |
| **Tech Lead** | = ENG-2 (wears both hats) | — | Architecture decisions (ADRs), gate sign-off with PM |

**RACI on key decisions:**

| Decision | PM | Tech Lead | ENG | SME |
|---|---|---|---|---|
| Gate pass/fail | A | R | C | C |
| Scope cut within a phase | A | C | C | I |
| Architecture / model choice (ADR) | C | A/R | R | I |
| Golden-set content | C | C | I | A/R |
| Risk acceptance / non-goal change | A | C | I | C |

*(A=Accountable, R=Responsible, C=Consulted, I=Informed)*

---

## 3. Capacity & velocity assumptions

- **Effective engineering capacity:** ~2.5 FTE. Using 8-point/engineer-week → ~40 pts/sprint nominal, discounted to **~26 pts/sprint** for meetings, ramp, infra friction, and the always-on eval lane.
- **Sprint 0 (ramp)** runs at reduced velocity (~16 pts): environment, access, corpus access, tooling.
- **SME is the scarcest resource** — golden-set authoring and gate reviews are scheduled explicitly so they never block a gate at the last minute.
- **Buffer:** every phase carries one explicit slack item; ~15% of capacity is unplanned-work reserve.

---

## 4. Ceremonies & artifacts

| Ceremony | Cadence | Output |
|---|---|---|
| Sprint planning | Day 1 of sprint | Committed sprint backlog (pointed) |
| Daily standup | Daily, 15 min | Blockers surfaced |
| Backlog refinement | Mid-sprint, 1 hr | Next sprint's stories ready (DoR met) |
| **Gate review** | End of each phase | Gate pass/fail decision + scorecard artifact (PM+TL+SME) |
| Sprint review/demo | Day 10 | Working increment demoed to stakeholders |
| Retro | Day 10 | 1–3 process actions |
| **Eval triage** (Scrum-ban lane) | Weekly | Failed-query post-mortems → golden-set additions |

**Definition of Ready (DoR):** story has acceptance criteria, is pointed, dependencies identified, test approach noted, and (if it touches retrieval/generation) a golden-set query class it should move.

**Definition of Done (DoD):** code merged behind config flags; unit/integration tests pass; tracing emits for the new path; golden-set eval run and delta recorded; docs/ADR updated; demoable.

---

## 5. Milestones & gates (the spine)

| Milestone | Sprint | Gate | Stakeholder-visible outcome |
|---|---|---|---|
| M0 Kickoff & infra ready | S0 | — | Dev env, corpus slice, golden-set framework live |
| M1 Basic RAG baseline | S1 | **G1** | Analyst asks 10 Qs live; citations open correct passages; baseline scorecard |
| M2 Production-grade retrieval | S2–S3 | **G2** | Recall@5 ≥0.70, QF ≥72%, citation precision ≥80%; ablation report |
| M3 Multimodal coverage | S4–S6 | **G3** | Scans/audio/video answerable; media citations resolve; bake-off ADR |
| M4 Graph & entity intelligence | S7–S8 | **G4** | Entity pages, multi-hop ≥50%, global ≥60%, router ≥85% |
| M5 Agentic + verified answers | S9–S10 | **G5** | Investigate mode, claim tables, abstention ≥80%, unsupported ≤5% |
| M6 Self-improving & handover | S11–S12 | **G6** | CI eval gating, 3 post-mortems, fine-tuned grader, rebuild-from-raw test |

---

## 6. Cross-phase workstreams (run continuously, not per-sprint)

| Workstream | Owner | Runs |
|---|---|---|
| **Evaluation & golden set** | ENG-2 + SME | S0 → end (grows each phase) |
| **Observability & tracing** | ENG-1 | S0 → end |
| **Infra & deployment (IaC)** | ENG-1 | S0 → end |
| **Risk burndown** | PM | every sprint (top-10 register reviewed) |
| **Security/sovereignty checks** | ENG-1 + PM | each gate (no-egress verification, license inventory) |

---

## 7. Sprint-by-sprint backlog

Story format: **[ID] Title** — *(pts)* — acceptance criterion. FR refs map to PRD. `M/S/C` = priority.

### Sprint 0 — Foundations & Ramp (Wks 1–2) · ~16 pts · Phase 1
**Sprint goal:** Everyone can build, deploy, and evaluate. Corpus slice in hand.

| ID | Story | Pts | Done when |
|---|---|---|---|
| S0-1 | Provision GPU + CPU servers; install Ollama + open-runtime containers | 3 | Qwen3-8B + BGE-M3 respond via local OpenAI-compatible endpoint |
| S0-2 | IaC repo (docker-compose → k8s-ready): OpenSearch, Qdrant, Neo4j, MinIO | 3 | `make up` brings the full stack up clean on a fresh host |
| S0-3 | Corpus census + stratified 1–5% slice selection (with SME) | 3 | Census report (formats×langs×collections); sampling script committed |
| S0-4 | Tracing/observability skeleton (Langfuse/Phoenix + OTel) | 2 | A dummy request produces an end-to-end trace |
| S0-5 | Golden-set framework + schema + 10 seed questions (with SME) | 3 | JSONL schema + harness stub runs, scores the 10 seeds |
| S0-6 | `pipeline.yaml` config-as-code scaffold + config-hash stamping | 2 | Two runs with same hash are reproducible |

**Risks:** hardware lead time (R5), corpus access delays. **Mitigation:** start S0-1/S0-3 day 1; if hardware slips, develop on smaller quantized models temporarily.

---

### Sprint 1 — Basic RAG E2E (Wks 3–4) · ~24 pts · Phase 1 → **Gate G1**
**Sprint goal:** A working cited chatbot over the text slice; baseline scorecard frozen.

| ID | Story | Pts | Done when |
|---|---|---|---|
| S1-1 | Ingestion: HTML + born-digital PDF → parser → chunker (FR-ING-1..5) | 5 | Slice ingested idempotently; chunks carry anchors |
| S1-2 | Embedding + Qdrant index; dense ANN retrieval API v0 (FR-RET-1,10) | 4 | `POST /v1/retrieve` returns ranked chunks w/ filters |
| S1-3 | Cited generation: evidence-only, insufficient-evidence path (FR-GEN-1..3) | 4 | Every factual sentence cited; gaps → "insufficient evidence" |
| S1-4 | Minimal chat UI + citation drill-down (text) + feedback (FR-UI-1,2,7) | 5 | Click citation → highlighted source passage |
| S1-5 | Golden set v1 to full Phase-1 count; baseline scorecard (FR-EVAL-1) | 3 | Scorecard artifact committed; all §3.2 metrics computed |
| S1-6 | Parse-quality sampling job (FR-ING-15) | 2 | 1% sample renders original-vs-extracted for review |
| — | **Gate G1 review** | — | Pipeline E2E + baseline published + golden set v1 frozen |

---

### Sprint 2 — Hybrid Retrieval (Wks 5–6) · ~26 pts · Phase 2
**Sprint goal:** BM25 + dense + RRF fusion + contextual chunking; retrieval recall jumps.

| ID | Story | Pts | Done when |
|---|---|---|---|
| S2-1 | BM25 in OpenSearch over chunks + contextual headers (FR-RET-2, FR-ING-6) | 5 | BM25 search live; headers prepended pre-index |
| S2-2 | RRF fusion + early metadata filters (FR-RET-3) | 4 | Fused candidate list; fusion weights config-driven |
| S2-3 | SLM query rewrite + alias expansion (FR-RET-5) | 4 | Rewritten query shown in trace; alias table seeded |
| S2-4 | Ablation harness: dense/bm25/hybrid configs (FR-EVAL-3) | 4 | One command diffs ≥3 config scorecards |
| S2-5 | UI filters panel (collection/date/type/lang) (FR-UI-3) | 3 | Filters flow through to retrieval |
| S2-6 | Context assembly: dedup, group-by-source, token budget (FR-GEN-6) | 4 | Assembled context respects hard token cap |
| — | Slack/buffer | 2 | — |

---

### Sprint 3 — Reranking + G2 (Wks 7–8) · ~26 pts · Phase 2 → **Gate G2**
**Sprint goal:** Cross-encoder reranking; hit production-grade retrieval; close Phase 2.

| ID | Story | Pts | Done when |
|---|---|---|---|
| S3-1 | Cross-encoder reranker service (BGE-reranker-v2) (FR-RET-4) | 5 | Top-200→top-20 rerank; scores in trace |
| S3-2 | Latency optimization + Fast-mode p95 budget (NFR-1) | 4 | p95 ≤ 6 s measured under load |
| S3-3 | Retrieval API hardening: versioned, documented (FR-RET-10) | 3 | OpenAPI spec; all consumers use it only |
| S3-4 | Ablation report (≥4 configs) + tuning fusion/rerank params | 4 | Report committed; params chosen on held-out split |
| S3-5 | Latency/throughput dashboard (NFR-11) | 3 | Grafana board live for G2 metrics |
| S3-6 | Golden-set growth + eval triage lane operational (FR-EVAL-5) | 3 | Weekly post-mortem producing golden-set items |
| — | **Gate G2 review** | — | Recall@5≥0.70, QF≥72%, citation prec≥80%, p95≤6s |
| — | Buffer | 4 | — |

---

### Sprint 4 — Multimodal: translate-to-text (Wks 9–10) · ~26 pts · Phase 3
**Sprint goal:** Audio/video/scans/images become answerable via the text pipeline.

| ID | Story | Pts | Done when |
|---|---|---|---|
| S4-1 | OCR path for scans + page-render generation (FR-ING-7) | 5 | Scanned PDFs searchable; OCR confidence stored |
| S4-2 | ASR (WhisperX) + diarization + timestamped chunks (FR-ING-8) | 5 | Audio answerable; citations carry mm:ss + speaker |
| S4-3 | Video keyframe + VLM caption pipeline (FR-ING-9) | 4 | Video frames captioned & indexed w/ timestamps |
| S4-4 | Image caption + VLM-OCR + EXIF (FR-ING-10) | 4 | Screenshots/images searchable |
| S4-5 | Media citation drill-down: audio seek / page region (FR-UI-2 media) | 5 | Click media citation → player seeks / page highlights |
| S4-6 | Golden set v2: +QV/QA questions (with SME) | 3 | ≥30 visual + ≥20 audio questions added |

**Risk:** ASR/OCR quality on noisy multilingual media (R9). **Mitigation:** language-ID routing + confidence flags; degraded-media expectations communicated to stakeholders.

---

### Sprint 5 — Multimodal: visual-native retrieval (Wks 11–12) · ~26 pts · Phase 3
**Sprint goal:** ColQwen page-image retrieval; decide per-media-type strategy by evidence.

| ID | Story | Pts | Done when |
|---|---|---|---|
| S5-1 | ColQwen2.5 visual index + two-stage search (FR-ING-11, FR-RET-6) | 6 | Page-image retrieval live; coarse→MaxSim rerank |
| S5-2 | Cross-modal RRF fusion (text + visual candidates) | 4 | Fused multimodal candidate list before final rerank |
| S5-3 | VLM answering over retrieved pages + region citations (FR-GEN-4) | 5 | VLM answers cite doc+page(+region) |
| S5-4 | Chart/number OCR cross-check safeguard (FR-GEN-5) | 4 | Numeric visual reads cross-checked; disagreements flagged |
| S5-5 | **Bake-off:** OCR-text vs ColQwen vs fused on scan slice → ADR | 4 | ADR records per-media-type routing decision |
| S5-6 | Chart-QA mini-eval set | 3 | Eval catches VLM number hallucination |

---

### Sprint 6 — Phase 3 close + Graph foundations (Wks 13–14) · ~26 pts · Phase 3 → **Gate G3** / Phase 4 start
**Sprint goal:** Close multimodal gate; begin entity extraction & graph.

| ID | Story | Pts | Done when |
|---|---|---|---|
| S6-1 | G3 hardening: regression check on text classes (≤2pt) | 3 | No text regression; QV≥60%, QA≥55% |
| S6-2 | NER + relation extraction w/ pinned prompt (FR-ING-12) | 5 | Entities/relations extracted; prompt version-tagged |
| S6-3 | Entity resolution: deterministic keys + embedding + aliases (FR-ING-13) | 5 | Canonical IDs; merges reversible & logged |
| S6-4 | Knowledge graph persisted in Neo4j w/ provenance (FR-ING-14) | 5 | Every edge → source chunk_ids |
| S6-5 | ER evaluation on labeled entity sample | 3 | Merge precision/recall report |
| — | **Gate G3 review** | — | Media classes pass; bake-off ADR signed |
| — | Buffer | 5 | — |

---

### Sprint 7 — GraphRAG retrieval (Wks 15–16) · ~26 pts · Phase 4
**Sprint goal:** Local + global graph search; query router.

| ID | Story | Pts | Done when |
|---|---|---|---|
| S7-1 | Graph local search: entity-link → k-hop → provenance chunks (FR-RET-7) | 5 | Entity queries traverse graph + join rerank |
| S7-2 | LazyGraphRAG-style global search (budgeted) (FR-RET-8) | 6 | Global/thematic queries answered via on-demand summaries |
| S7-3 | Query router SLM classifier (FR-RET-9) | 5 | Routes to strategy; low-conf → hybrid default |
| S7-4 | Router labeled set (≥200 queries) + router eval | 3 | Confusion matrix; ≥85% accuracy target measured |
| S7-5 | Graph answer path explanation + citations (FR-GEN-7) | 4 | Answers show "A→X→B" with per-edge sources |
| — | Buffer | 3 | — |

---

### Sprint 8 — Graph UI + G4 (Wks 17–18) · ~26 pts · Phase 4 → **Gate G4**
**Sprint goal:** Analyst-facing entity/graph/timeline; close Phase 4.

| ID | Story | Pts | Done when |
|---|---|---|---|
| S8-1 | Entity page: aliases, attributes, relations, mentions (FR-UI-4) | 5 | Entity pages render for analyst-chosen entities |
| S8-2 | Interactive graph neighborhood explorer (FR-UI-5) | 5 | Expand/filter nodes; answer-path highlight |
| S8-3 | Timeline retrieval + view (FR-RET-11, FR-ING events) | 5 | Event nodes sorted; "evolution over time" answerable |
| S8-4 | Incremental ingestion (no full rebuild) (FR-ING-16) | 4 | New files update indexes + append to graph |
| S8-5 | Golden set v3: +QH/QG/timeline (with SME) | 3 | ≥35 multi-hop + ≥30 global added |
| — | **Gate G4 review** | — | QH≥50%, QG≥60%, router≥85%, ER merge≥90% |
| — | Buffer | 4 | — |

---

### Sprint 9 — Agentic orchestration (Wks 19–20) · ~26 pts · Phase 5
**Sprint goal:** Investigate mode: planner + bounded loop + CRAG grading.

| ID | Story | Pts | Done when |
|---|---|---|---|
| S9-1 | Fast vs Investigate mode split + orchestrator (LangGraph, wrapped) (FR-AGT-1) | 5 | Mode selectable; orchestrator behind own API |
| S9-2 | Planner: decompose → per-subq strategy → plan object (FR-AGT-2) | 5 | Plan streamed to user |
| S9-3 | Bounded execution: iteration/token/wallclock caps (FR-AGT-3) | 4 | Caps enforced; "budget exhausted" path works |
| S9-4 | CRAG retrieval grader (SLM) + retry logic (FR-AGT-4) | 5 | Insufficient evidence → rewrite/broaden ≤2 retries |
| S9-5 | Full agent trace persisted + viewer (FR-AGT-5, FR-UI-8) | 4 | Plan, tool calls, verdicts, budgets viewable |
| — | Buffer | 3 | — |

---

### Sprint 10 — Verification & confidence + G5 (Wks 21–22) · ~26 pts · Phase 5 → **Gate G5**
**Sprint goal:** Claim-level grounding, confidence, abstention; close Phase 5.

| ID | Story | Pts | Done when |
|---|---|---|---|
| S10-1 | Claim extraction + claim→citation mapping (FR-VER-1) | 4 | Draft split into atomic claims |
| S10-2 | Claim grounding check (NLI/judge ≠ generator) (FR-VER-2) | 5 | Unsupported claims removed/flagged |
| S10-3 | Chain-of-verification for Investigate (FR-VER-3) | 4 | Verification questions answered pre-finalize |
| S10-4 | Per-claim confidence + conflict surfacing + credibility store (FR-VER-4,5,6) | 5 | Confidence H/M/L w/ breakdown; conflicts shown |
| S10-5 | Investigate UX: plan→progress→claim table→dissent (FR-UI-6) | 4 | Full Investigate answer view |
| S10-6 | Red-team set (50 adversarial) + indirect-injection tests | 3 | Abstention/correction + injection-resistance measured |
| — | **Gate G5 review** | — | QU≥80%, unsupported≤5%, QH≥65%, Investigate p95≤4min |
| — | Buffer | 1 | — |

---

### Sprint 11 — Reflection & self-correction (Wks 23–24) · ~26 pts · Phase 6
**Sprint goal:** Adaptive retrieval + reflective critic + continuous eval in CI.

| ID | Story | Pts | Done when |
|---|---|---|---|
| S11-1 | Adaptive retrieval complexity classifier (FR-VER-7) | 4 | Skip/single/iterate decisions logged |
| S11-2 | Reflective critic pass + revise/trim loop (FR-VER-8) | 5 | Segment-level support/usefulness checks |
| S11-3 | CI eval gate: regression >2pt blocks merge (FR-EVAL-4) | 4 | PR touching retrieval/gen runs golden set in CI |
| S11-4 | Post-mortem workflow → golden-set growth (3 cycles started) (FR-EVAL-5) | 4 | First post-mortem cycle completed |
| S11-5 | Drift dashboard (FR-EVAL-6) | 3 | Score dists, abstention rate, per-class accuracy over time |
| S11-6 | NLI checker calibration vs 200 human-labeled pairs | 3 | ≥85% agreement measured |
| — | Buffer | 3 | — |

---

### Sprint 12 — Hardening, fine-tune & handover + G6 (Wks 25–26) · ~24 pts · Phase 6 → **Gate G6**
**Sprint goal:** Close the loop; prove rebuild-from-raw; ship POC report.

| ID | Story | Pts | Done when |
|---|---|---|---|
| S12-1 | Complete 3rd post-mortem cycle; golden-set final size | 3 | 3 cycles done; golden set at final counts |
| S12-2 | Fine-tune SLM relevance grader (LoRA) + before/after ablation | 5 | Grader FT ablation report committed |
| S12-3 | Rebuild-from-raw test: new engineer rebuilds all indexes from docs | 4 | Indexes rebuilt using docs alone (timed) |
| S12-4 | Security/sovereignty final audit: no-egress + license inventory | 3 | Verified zero external calls; license list complete |
| S12-5 | POC final report: metric trajectory P1→P6, cost/query, prod scaling plan | 5 | Report references companion architecture doc |
| S12-6 | Stakeholder demo + go/no-go-to-production recommendation | 2 | Demo delivered; recommendation documented |
| — | **Gate G6 review** | — | CI gating live, 3 post-mortems, grader FT, rebuild test pass |
| — | Buffer | 2 | — |

---

## 8. Dependency map (what blocks what)

```
S0 infra ──► S1 basic RAG ──► S2 hybrid ──► S3 rerank (G2)
                  │                              │
                  └► golden set v1 ──────────────┴──► grows every sprint
S3 ──► S4 translate-to-text ──► S5 visual-native ──► S6 (G3) + extraction
                                                          │
S6 extraction+ER ──► S7 graph retrieval ──► S8 graph UI (G4)
                          │
S8 ──► S9 agentic ──► S10 verification (G5) ──► S11 reflection ──► S12 (G6)

Always-on lanes (S0→S12):  eval harness · tracing · IaC · risk burndown
```

**Hard dependencies (cannot parallelize):**
- Retrieval API (S1) blocks everything downstream — it is the contract.
- Entity extraction + ER (S6) blocks all graph work (S7–S8).
- A solid retrieval core (G2) must precede agents (S9) — agents over a weak retriever are just slower wrongness.
- Golden set must exist (S1) before any gate can be judged.

**Safe to parallelize:**
- UI work (ENG-3) runs one slice behind backend throughout.
- Multimodal ingestion sub-paths (OCR / ASR / video / image) in S4 are independent and split across the pair.
- Tracing/IaC/eval lanes run continuously alongside features.

---

## 9. Risk burndown schedule (top risks tied to sprints)

| Risk (from PRD §11) | Burned down by | Sprint |
|---|---|---|
| R1 Parsing garbage | Census + sampling + per-format fixtures **before** Phase 2 | S0–S1 |
| R2 Golden set too easy | SME-authored hard cases + held-out split early | S1 |
| R3 Entity drift | ER eval on labeled sample | S6 |
| R4 VLM chart hallucination | OCR cross-check + chart-QA eval | S5 |
| R5 Agent cost/latency blowout | Hard budgets shipped with the loop | S9 |
| R6 LLM-judge drift | Human calibration set | S10–S11 |
| R7 Indirect prompt injection | Adversarial suite | S10 |
| R8 Model/tool churn | Service boundaries + config-as-code from day 1 | S0 |
| R9 ASR/OCR noisy media | Confidence flags + lang routing | S4 |
| R10 Scope creep | Non-goals enforced via change-control | every sprint (PM) |

PM reviews the full top-10 register at each sprint review; any risk trending red triggers a mitigation story in the next sprint.

---

## 10. Scope-control policy (how we protect the gates)

When a sprint is over capacity or a gate is at risk, cuts happen in this fixed order:
1. Drop `Could` items.
2. Drop/defer `Should` items to a "Phase N+1 backlog."
3. Narrow the corpus slice or query-class scope for the gate (with SME sign-off).
4. **Never** cut: the gate's `Must` requirements, the eval harness, tracing, or the sovereignty/no-egress checks.

Any change to PRD goals, non-goals, or gate thresholds requires PM + Tech Lead sign-off and a written justification (per PRD §0). New feature requests mid-phase go to the backlog, not the active sprint.

---

## 11. Stakeholder communication plan

| Audience | Cadence | Format | Content |
|---|---|---|---|
| Sponsor / leadership | End of each phase (gate) | 1-page + demo | Gate result, metric trajectory, cost, risks, next-phase ask |
| Analyst team (users) | Sprint reviews | Live demo | New capabilities; collect feedback into golden set |
| Engineering | Daily / sprint | Standup + board | Blockers, velocity, eval deltas |
| Security / compliance | Each gate | Audit note | No-egress verification, license inventory, data-handling |

**Reporting metric that travels everywhere:** the golden-set scorecard trajectory (accuracy by query class, citation precision, abstention quality, p95 latency, compute/query) — one chart, updated every sprint, is the single source of truth for "are we winning."

---

## 12. What "success" looks like at the end of S12

- All six gates passed on the held-out golden set.
- An analyst can run both Fast and Investigate modes over the full corpus slice, with citations that resolve to page/timestamp, confidence scores, and honest abstention.
- The system is fully open-source, self-hosted, zero-egress — verified by audit.
- A new engineer can rebuild every index from raw corpus using the docs alone.
- The continuous-eval loop is live, so the system keeps improving after the POC.
- A documented, evidence-backed **go/no-go recommendation for production**, with the metric trajectory and the companion production-architecture doc as the scaling blueprint, and the model-upgrade menu as the path to higher quality if/when data policy allows.

---

## Appendix A — Sprint summary at a glance

| Sprint | Weeks | Phase | Theme | Gate | ~Pts |
|---|---|---|---|---|---|
| S0 | 1–2 | 1 | Foundations & ramp | — | 16 |
| S1 | 3–4 | 1 | Basic RAG E2E | **G1** | 24 |
| S2 | 5–6 | 2 | Hybrid retrieval | — | 26 |
| S3 | 7–8 | 2 | Reranking | **G2** | 26 |
| S4 | 9–10 | 3 | Multimodal (to-text) | — | 26 |
| S5 | 11–12 | 3 | Visual-native retrieval | — | 26 |
| S6 | 13–14 | 3→4 | Close MM + graph foundations | **G3** | 26 |
| S7 | 15–16 | 4 | GraphRAG retrieval | — | 26 |
| S8 | 17–18 | 4 | Graph UI + timeline | **G4** | 26 |
| S9 | 19–20 | 5 | Agentic orchestration | — | 26 |
| S10 | 21–22 | 5 | Verification & confidence | **G5** | 26 |
| S11 | 23–24 | 6 | Reflection & self-correction | — | 26 |
| S12 | 25–26 | 6 | Hardening, FT & handover | **G6** | 24 |

*Velocity and point estimates are planning priors; re-baseline after S1 actuals, per §3.*
