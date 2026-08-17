# DFS — Defence Data Verification & Factuality System
## 00 · Product Requirements

**Status:** design, pre-implementation · **Version:** 0.1 · **Date:** 2026-08-17

---

## 1. Problem

Defence information arrives from mixed-quality sources (ministry press offices, primes, trade press, wires, think tanks, OSINT accounts, vendor marketing, leaks, forums). Downstream consumers treat whatever lands in the knowledge base as fact. Three failure modes destroy that KB:

| Failure | Mechanism | Cost |
|---|---|---|
| **Echo** | One outlet publishes a wrong number; 40 outlets copy it. Naive corroboration reads 40 confirmations. | Confidently wrong at scale. |
| **Scoop loss** | A low-reputation source is genuinely first and correct. A trust gate discards it. | Lost intelligence + the source can never earn reputation (self-sealing). |
| **False conflict** | "200 km / 210 km / 230 km" treated as disagreement when they are measurements of *different quantities*. | The system "resolves" incompatible numbers into a value nobody asserted. |

DFS exists to make each of these structurally impossible, not statistically unlikely.

## 2. Objective

Convert raw incoming documents into **span-grounded, qualified, provenance-tracked claims**, compute a **calibrated belief** over each fact, and publish to the KB with **status, confidence, evidence and reason attached** — never a bare value.

**Primary success metric is calibration, not accuracy.** A system that says 0.7 and is right 70% of the time is usable. A system that says 0.95 and is right 80% of the time is dangerous.

## 3. Users

| User | Needs |
|---|---|
| Analyst | Fast triage of high-impact/low-confidence claims; drill-down to source spans; ability to overturn and have that overturn count. |
| Downstream KB consumer (API) | A value that *always* arrives with status + confidence + as-of date. |
| Source curator | Which sources are earning trust, in which domains, on which claim types, with what sample size. |
| Auditor | Full reconstruction of why any stored fact is believed, at any past point in time. |

## 4. Scope

**In scope (v1):** ingest, claim extraction, normalisation, entity/variant resolution, provenance & independence clustering, evidence assessment, belief computation, publication decision, KB reconciliation, outcome ledger, analyst review queue.

**Out of scope (v1):** source *collection* / crawling (assumed upstream), imagery/SIGINT exploitation, classification-level handling beyond spill detection & quarantine, natural-language querying of the KB, automated tasking of collection.

**Explicit non-goals:** DFS never asserts truth. It asserts *a calibrated belief with its evidence*. It never deletes evidence. It never computes a value no source asserted.

## 5. Functional requirements

| ID | Requirement | Priority |
|---|---|---|
| FR-1 | Every stored claim carries byte-offset spans into its source document; quoted numbers must string-match the source text. | MUST |
| FR-2 | Every stored claim carries a qualifier vector with **explicit UNKNOWNs**. Imputed qualifiers are stored as flagged hypotheses, never as assertions. | MUST |
| FR-3 | Corroboration is counted in **independent provenance clusters**, never in source or article count. | MUST |
| FR-4 | No single source, at any reputation, may push a claim across the ACCEPT threshold alone. | MUST |
| FR-5 | The evidence layer is append-only. Nothing is ever deleted or overwritten, including quarantined disinformation. | MUST |
| FR-6 | The belief layer is *derived* and fully recomputable from the evidence layer + algorithm version + config hash. | MUST |
| FR-7 | Facts are bitemporal: valid-time (when true of the world) and record-time (when we learned it). | MUST |
| FR-8 | Source competence is scored per (source × domain × claim-type) as a posterior with sample size, not a scalar. | MUST |
| FR-9 | The only stage permitted to hard-reject input is relevance triage (G1), and it samples its rejects for QA. | MUST |
| FR-10 | No LLM output is written to the KB without passing a deterministic validator. No confidence number is ever produced by an LLM. | MUST |
| FR-11 | Retraction or source compromise cascades: every fact derived from the affected claims is recomputed. | MUST |
| FR-12 | Every API read returns `status` and `confidence` alongside `value`. A value-only read path must not exist. | MUST |
| FR-13 | Claims that cannot be made comparable terminate in an honest `not_comparable` state rather than a forced resolution. | MUST |
| FR-14 | High-impact claims route to human review regardless of computed confidence, below configured bars. | MUST |
| FR-15 | Source reputation credit is weighted by novelty at publication time (first-and-right ≫ fortieth-and-right). | SHOULD |
| FR-16 | Suspected coordinated campaigns (synchronous, uncited, multi-source) raise a flag rather than a corroboration boost. | SHOULD |

## 6. Non-functional requirements

| ID | Requirement |
|---|---|
| NFR-1 | Belief computation is **deterministic**: same evidence + same config hash → bit-identical output. Required for audit and regression. |
| NFR-2 | Throughput target v1: 50k documents/day ingest, 500k claims/day. Belief recompute is not on the hot read path (materialised). |
| NFR-3 | Latency: ingest→publication decision p95 < 15 min for routine claims. Human review SLA: critical 4h, sensitive 24h. |
| NFR-4 | Full replay of the belief layer over the frozen evidence set completes in < 6h (regression gate for any algorithm change). |
| NFR-5 | Every score displayed anywhere must display its sample size `n` and the cell's resolution rate. No naked decimals. |
| NFR-6 | Analyst review queue is ordered by impact × uncertainty, not FIFO, and has a defined drain SLA — quarantine must not be where claims go to die. |

## 7. Input contract

Ingest accepts a **Document**. DFS extracts claims from it; callers do not submit claims directly (a claim-submission API is a v2 item for analyst-authored assertions).

```jsonc
{
  // ── REQUIRED ────────────────────────────────────────────────
  "doc_id":        "sha256:...",         // content hash, dedup key
  "source_id":     "src:janes",          // must exist in source registry (see FR: cold start)
  "retrieved_at":  "2026-08-17T09:12:00Z",
  "content_type":  "text/html",
  "raw":           "<full original bytes/text — stored verbatim, never normalised in place>",

  // ── STRONGLY EXPECTED (degrades quality if absent) ──────────
  "published_at":  "2026-08-16T18:00:00Z", // DISTINCT from retrieved_at — copy-chain ordering depends on it
  "url":           "https://...",
  "title":         "...",
  "language":      "en",

  // ── OPTIONAL ────────────────────────────────────────────────
  "authors":       ["..."],               // byline sharing is an independence signal
  "citations":     [{"text":"according to Reuters","target_url":"..."}], // extracted if absent
  "wire_credit":   "AP",                  // syndication marker; high-value independence signal
  "attachments":   [{"kind":"pdf|image","hash":"...","uri":"..."}],
  "collection_ctx": {"tasking_id":"...","collector":"...","notes":"..."},
  "handling":      {"classification":"UNCLASS","releasability":"...","jurisdiction":"..."}
}
```

**Required-field rationale.** `published_at` is separate from `retrieved_at` because copy-chain direction is inferred from publication ordering; collapsing them destroys echo detection permanently and cannot be reconstructed later. `raw` is stored verbatim because span grounding (FR-1) validates against the original bytes.

**Rejection at intake (structural only):** missing `doc_id`/`source_id`/`raw`, unknown `source_id`, or content hash already present (dedup → link, do not re-ingest).

## 8. Output contract

The KB read API returns a **Fact belief view**. Never a bare value (FR-12).

```jsonc
{
  "fact_id": "fact:radar-x/detection_range",
  "entity":  {"id":"ent:radar-x","label":"Radar X","variant":"Block 2","resolved_via":"alias_table"},
  "attribute": "detection_range",
  "qualifiers": {                        // what this value is actually about
    "quantity_type": "detection_range",
    "target_rcs_m2": 1.0,
    "target_altitude_m": {"value": null, "state": "UNKNOWN"},
    "mode": "volume_search",
    "pd": 0.9,
    "environment": {"value": null, "state": "UNKNOWN"},
    "number_provenance": "acceptance_test"
  },
  "value":      {"magnitude": 200, "unit": "km", "granularity": 10},
  "status":     "consensus_with_dissent", // corroborated | single_source | consensus_with_dissent
                                          // | disputed | not_comparable | quarantined_only | refuted
  "confidence": 0.78,                     // calibrated P(claim true) — deterministic, reproducible
  "support": {
    "independent_clusters": 3,            // NOT article count
    "n_eff": 2.4,
    "raw_source_count": 17,               // shown so the gap between 17 and 2.4 is visible
    "strongest_evidence": "primary_document"
  },
  "dissent": [
    {"value":{"magnitude":230,"unit":"km"},"clusters":1,"weight":0.19,
     "sources":["src:vendor-brochure"],"note":"manufacturer figure, unqualified"}
  ],
  "contradictions": [
    {"kind":"qualifier_mismatch","with_claim":"clm:...","detail":"RCS unstated; not comparable"}
  ],
  "validity": {"valid_from":"2019-06-01","valid_to":null,"as_of":"2026-08-17"},
  "provenance": {
    "supporting_claims": ["clm:a1","clm:b7","clm:c2"],
    "clusters": [{"cluster_id":"pc:1","members":["clm:a1","..."],"root":"clm:a1","rho_method":"citation+minhash"}],
    "algorithm_version": "belief-v0.4.1",
    "config_hash": "sha256:...",
    "computed_at": "2026-08-17T09:31:00Z"
  },
  "decision": {
    "outcome": "PROVISIONAL",
    "reason_codes": ["SINGLE_PRIMARY_CHAIN","DISSENT_PRESENT"],
    "reason": "One primary-document cluster (acceptance test) plus two trade-press clusters agree at 200 km; one unqualified vendor figure at 230 km retained as dissent. No second independent primary chain, so below ACCEPT bar.",
    "human_reviewed_by": null
  }
}
```

Every field above is machine-generated and reproducible. `reason` is the only free text and is templated from `reason_codes` — not LLM prose.

## 9. Acceptance criteria for the design phase

- [ ] Every gate has Input → Checks → Logic → Score → Threshold → Outcome specified numerically (see `01-architecture.md`).
- [ ] Source scoring produces a number *with* a sample size and a resolution rate, in every path (`02-source-model.md`).
- [ ] The conflict algorithm has an ordered discriminating-test sequence, and no branch of it averages values (`03-conflict-and-belief.md`).
- [ ] Every architectural decision has an alternatives log with a stated reason (`05-decision-log.md`).
- [ ] Roadmap milestones each have a numeric exit metric (`04-data-eval-roadmap.md`).
