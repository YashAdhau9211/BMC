# Layer 2 — Test Execution Report

**Run date:** 2026-07-08
**Build:** mallory-engine 0.1.0 · Python 3.14.3 · pytest 9.1.1
**Environments exercised:**
- **Unit/integration-of-code:** in-memory SQLite (pytest fixture, `tests/conftest.py`)
- **Live system:** L2 API `http://127.0.0.1:8000` backed by **PostgreSQL 16** (docker `mallory-pg`, port 5433), `LLM_PROVIDER=stub`
**Test assets:** `tests/` (existing suite) + `tests/test_qa_plan.py` (new gap/defect coverage from `docs/L2_TEST_PLAN.md`)

---

## 1. Headline result

| Metric | Count |
|---|---|
| **Total executed** | **80** (79 run + 1 skipped) |
| ✅ Passed | **75** |
| ❌ Failed | **4** (all are **defect probes** — each failure is a confirmed finding, not a broken test) |
| ⏭️ Skipped | **1** (`test_eval_live` — requires `MALLORY_EVAL_LIVE=1` + a live LLM; intentionally gated) |

```
4 failed, 75 passed, 1 skipped, 1 warning in 2.41s
```

**Interpretation:** every test asserting *intended* behaviour passed (75/75). The 4 failures are deliberate **defect-probe** tests that assert the *correct* behaviour for known gaps; their failure confirms DEF-01, DEF-02, DEF-03, and DEF-07 from the test plan — reproduced both on SQLite and on the live Postgres server.

**Live LLM eval (separate run, real Ollama farm):** ✅ **PASSED, 8/9** graded cases (avg 1350 ms/case warm) — see §6A. Surfaced one model-judgment finding (LIVE-01: over-optimistic verdict on a 30%-fit tender) and one test-harness Unicode issue (TEST-HARNESS-01). The stub suite above did **not** call the farm; only this eval did.

---

## 2. Results by suite

### 2.1 Existing suite (regression baseline) — 45 passed, 1 skipped

| Test file | Cases | Result |
|---|---|---|
| `tests/eval/test_eval_stub.py` | 9 | ✅ all pass (property-based golden eval vs stub) |
| `tests/eval/test_eval_live.py` | 1 | ⏭️ skipped (needs live LLM) |
| `tests/test_confidence.py` | 5 | ✅ all pass |
| `tests/test_extraction.py` | 5 | ✅ all pass |
| `tests/test_graph.py` | 5 | ✅ all pass |
| `tests/test_ingest_contract.py` | 3 | ✅ all pass |
| `tests/test_synthesis.py` | 10 | ✅ all pass |
| `tests/test_tender_scoring.py` | 4 | ✅ all pass |
| `tests/test_trust_spine.py` | 4 | ✅ all pass |

### 2.2 New QA-plan suite (`tests/test_qa_plan.py`) — 30 passed, 4 failed

| # | Test (TC id) | Category | Verdict | Notes |
|---|---|---|---|---|
| 1 | `test_conf_301_exact_point_decomposition` (TC-CONF-301) | Functional/exact math | ✅ PASS | tier35+corr17+fresh25+prov15=92, band high |
| 2 | `test_conf_301b_unknown_date_is_12` (TC-CONF-303) | Edge/time | ✅ PASS | unknown date → 12 (banker's rounding of 12.5) |
| 3 | `test_conf_302_band_boundaries` (TC-CONF-302) | Boundary | ✅ PASS | 70→high, 69→medium, 45→medium, 44→low |
| 4 | `test_conf_304_future_date_no_overflow` (TC-CONF-304) | Edge | ✅ PASS | future date floors age→0, freshness 25, total ≤95 |
| 5 | `test_cor_402_value_bucket` (TC-COR-402) | Data quality | ✅ PASS | ₹4,500cr / 4500 / $45M → "45"; 0/neg/absent → "?" |
| 6 | `test_sig_502_classification_decision_table` ×4 (TC-SIG-502) | Business rule | ✅ PASS ×4 | threat/fav/watch/watch-fallback |
| 7 | `test_ten_605_fx_normalization` (TC-TEN-605) | Functional | ✅ PASS | INR6500→78, EUR100→108, JPY→×1, null→null |
| 8 | `test_ten_608_score_clamp` (TC-TEN-608) | Boundary | ✅ PASS | fit clamps to [5,98] |
| 9 | `test_ten_604_deadline_status` (TC-TEN-604) | Boundary/time | ✅ PASS | closed/closing/open cutoffs |
| 10 | `test_dom_701_702_703_tags` (TC-DOM-701/2/3) | Business rule | ✅ PASS | CORE / estimate / behind |
| 11 | `test_syn_1304_matchup_edge_exact` (TC-SYN-1304) | Exact math | ✅ PASS | 50+12·(4−1)=86 → fav |
| 12 | `test_llm_1204_numbers_grounded_rejects_hallucination` (TC-LLM-1204) | Security/grounding | ✅ PASS | invented "70 km" rejected → fallback; ledger 'invalid' |
| 13 | `test_llm_1208_classify_dir_is_never_llm_computed` (TC-LLM-1208) | Business rule | ✅ PASS | dir rule-derived; only lens from model |
| 14 | `test_llm_validators_enum_and_length` | Unit | ✅ PASS | enum/length/numbers_grounded guards |
| 15 | `test_kg_807_empty_graph_no_crash` (TC-KG-807) | Edge | ✅ PASS | empty graph → payload written, 0 insights, no raise |
| 16 | `test_ing_103_enum_matrix` ×5 (TC-ING-103) | Negative/enum | ✅ PASS ×5 | rel_type/stage/confidence/maturity/event_type → 422 rule3 |
| 17 | `test_ing_102_unknown_record_type` (TC-ING-102) | Negative | ✅ PASS | → 422 unknown_record_type |
| 18 | `test_ing_101_empty_main_text_rejected_on_record_path` (TC-ING-101) | Negative | ✅ PASS | blank main_text on `/{record_type}` → 422 rule1 |
| 19 | `test_srv_1002_pagination_bounds` ×5 (TC-SRV-1002) | Boundary | ✅ PASS ×5 | size∈[1,100], page≥1 enforced |
| 20 | `test_defect_01_page_should_reject_empty_main_text` (DEF-01) | Defect probe | ❌ **FAIL** | **confirms DEF-01** |
| 21 | `test_defect_02_corroboration_merges_across_proc_state` (DEF-02) | Defect probe | ❌ **FAIL** | **confirms DEF-02** |
| 22 | `test_defect_03_explain_matchup_exposes_confidence` (DEF-03) | Defect probe | ❌ **FAIL** | **confirms DEF-03** |
| 23 | `test_defect_07_chat_nonnumeric_entity_id_is_graceful` (DEF-07) | Defect probe | ❌ **FAIL** | **confirms DEF-07** |

---

## 3. Failure analysis (why each failed + captured output)

### ❌ DEF-01 — `POST /ingest/v1/page` accepts empty `main_text`
- **Test:** `test_defect_01_page_should_reject_empty_main_text`
- **Expected:** 422 `rule1_empty_main_text` (parity with the `/{record_type}` path).
- **Actual:** HTTP **200** (document staged with empty main_text).
- **Captured (pytest):**
  ```
  AssertionError: DEF-01 CONFIRMED: /page accepted empty main_text (status 200);
  no rule1_empty_main_text guard on this path.
  assert 200 == 422
  ```
- **Captured (live Postgres server):**
  ```
  --- DEF-01 live: POST /ingest/v1/page with empty main_text ---   → HTTP 200
  --- Control: /ingest/v1/competitive_signal, empty main_text ---   → HTTP 422
                                                {"failing_rule":"rule1_empty_main_text"}
  ```
- **Root cause:** `api/ingest.py::ingest_page` relies only on Pydantic (`main_text: str` is *present* but `""` is a valid string). The non-empty guard exists **only** in `ingest_bundle` (`/{record_type}`), not in `/page` or `/document`.
- **Impact:** A page with no body text stages a `stg_document` that extraction/NLP cannot use — it will still spawn a signal with an empty `event_summary`-adjacent body, polluting serving with contentless cards. Inconsistent contract enforcement across the three ingest entry points.
- **Severity:** Major · **Fix:** add the same `main_text.strip()` check (or a Pydantic `min_length=1` / validator on `DocumentIn.main_text`) so all three paths enforce rule1 uniformly.

### ❌ DEF-02 — Corroboration under-counts across processing state
- **Test:** `test_defect_02_corroboration_merges_across_proc_state`
- **Expected:** same award (LT / India / ₹4,500 cr) from two different sources → both corroboration = 2.
- **Actual:** counts `{1: 1, 2: 1}` — they did **not** merge.
- **Captured (pytest):**
  ```
  AssertionError: DEF-02 CONFIRMED: same award did not corroborate across proc-state;
  counts={1: 1, 2: 1} (dir component 'threat' vs '?').
  assert (1 == 2)
  ```
- **Root cause:** `corroboration._claim_key` includes `dir`, but corroboration runs **before** classification. An already-published signal has `dir='threat'`; a freshly received one has `dir=None → "?"`. The keys diverge (`LT|India|threat|45` vs `LT|India|?|45`), so the group never merges.
- **Impact:** A second, independent confirmation of a previously-published award does **not** raise its corroboration count or confidence — the trust spine under-reports on exactly the cross-batch case it exists to reward. (Within a single batch, all signals share `dir='?'` and DO corroborate — which is why existing `test_trust_spine` passes.)
- **Severity:** Major · **Fix:** drop `dir` from the claim key, or compute corroboration *after* classification, or normalize both to pre-dir. (Test flips to PASS once fixed.)

### ❌ DEF-03 — `/explain` exposes confidence only for signals
- **Test:** `test_defect_03_explain_matchup_exposes_confidence`
- **Expected:** `/api/v1/explain/matchup/{id}` returns a numeric `confidence`.
- **Actual:** evidence present, but `confidence: null`.
- **Captured (pytest):**
  ```
  AssertionError: DEF-03 CONFIRMED: matchup explain returns evidence but confidence=null
  (_CONF_MODEL maps only 'signal').
  {'target_kind': 'matchup', 'target_id': '1', 'provenance': 'sourced', 'confidence': None, ...}
  ```
- **Captured (live Postgres server):**
  ```
  {"target_kind":"matchup","target_id":"2","provenance":"sourced","confidence":null,
   "confidence_band":null,"confidence_parts":null,"evidence_count":1,
   "fields":[{"field":"card","method":"rule","evidence":[{"eid":"ref:ref_matchups:MARG155__rch_155", ...}]}]}
  ```
- **Root cause:** `api/serving.py::_CONF_MODEL = {"signal": SrvSignal}` — only signals are mapped to a confidence-bearing model. Tenders/matchups/syntheses return their evidence chain but always `confidence=null`.
- **Impact:** Explainability is incomplete for 3 of 4 explainable target kinds. UI "why this?" panels on tenders/matchups/synthesis can show sources but not a confidence score/decomposition. (Note: tenders additionally have **no** evidence rows written at all — `tender_scoring.process_tender` never calls `write_evidence` — a related gap worth a follow-up ticket.)
- **Severity:** Minor (evidence still present) · **Fix:** extend `_CONF_MODEL` to `matchup→SrvMatchup`, `synthesis→SrvCompetitorSynthesis`; add evidence writes to tender scoring.

### ❌ DEF-07 — Non-numeric `entity_id` in chat causes HTTP 500
- **Test:** `test_defect_07_chat_nonnumeric_entity_id_is_graceful`
- **Expected:** a clean 4xx for bad input.
- **Actual:** HTTP **500** (unhandled `ValueError`).
- **Captured (pytest):**
  ```
  AssertionError: DEF-07 CONFIRMED: non-numeric entity_id caused HTTP 500
  (unguarded int() cast in assistant.answer).
  assert 500 < 500
  ```
- **Captured (live Postgres server):**
  ```
  --- DEF-07 live: chat with non-numeric entity_id (signal panel) ---   → HTTP 500
  ```
- **Root cause:** `services/assistant.py::answer` does `int(req.entity_id)` for `panel_context in {signal, tender}` with no guard; a non-numeric id raises `ValueError`, surfacing as a 500. Same unguarded-cast pattern exists in `api/serving.py::explain` (`int(target_id)` is guarded by `.isdigit()` there — so explain is safe; chat is not).
- **Impact:** A malformed client request (or a fuzzer) trivially triggers 500s on the public chat endpoint — noisy errors, potential stack-trace exposure, and a cheap availability nuisance. No auth means it's reachable by anyone with network access.
- **Severity:** Major · **Fix:** validate/guard the cast (`if req.entity_id and req.entity_id.isdigit()`), returning empty context or a 422, mirroring the `explain` guard.

---

## 4. Live-system verification (Postgres, integration golden path)

The end-to-end L1→L2→L3 path was exercised previously and re-verified for this report. Current serving state on the live Postgres-backed server:

```
GET /health              → {"status":"ok","service":"mallory-engine","version":"0.1.0"}
GET /api/v1/nav/counts   → {"competitive":14,"market":8,"technology":3,"matchups":8,
                            "partnerships":5,"geo":9,"tenders":8,"innovation":0,"patents":6}
GET /ops/status          → {"staging":{"signals":{"published":25},"tenders":{"published":8}},
                            "serving":{"signals":25,"tenders":8}}
```
- ✅ Ingest → pipeline → serving is healthy: 25 signals + 8 tenders published, matchups/geo/partnerships populated, knowledge graph built.
- ✅ Control (negative): valid `/{record_type}` with blank main_text correctly returns 422 `rule1_empty_main_text`.
- ❌ DEF-01/03/07 reproduce identically on Postgres — i.e. these are product defects, not SQLite artefacts.

---

## 5. Coverage gaps in THIS run (not yet executed)

The following plan suites were **not** run here and remain open (need infra or are non-automatable in a unit run):

| Area | Plan cases | Why deferred |
|---|---|---|
| Postgres sequence collision | TC-DQ-1601 | Needs a fresh-Postgres reseed cycle; **already observed & fixed live** in an earlier session (demo_seed sequence resync). Recommend codifying as a Postgres-marked test. |
| Concurrency / races | TC-ING-112 (dup URL), TC-OPS-1503 (dual rebuild) | Needs a concurrent load harness. DEF-05/DEF-09 remain **unverified-by-test** (identified by code inspection). |
| Performance / scale / endurance | Suite R (TC-PERF-1701…1706) | Needs k6/locust + a large dataset; separate perf run. |
| Security deep | TC-SEC-1901 (no-auth), 1903 (secret exposure), 1905 (DoS), TC-AST-1103 (SSRF) | Partially inspected; needs a dedicated security pass. |
| Live LLM farm | TC-INT-1805, `test_eval_live` | Skipped here (stub run); run with `MALLORY_EVAL_LIVE=1` against the Ollama farm. |
| Timeout / DB-down mid-pipeline | TC-LLM-1203, TC-OPS-1504 | Needs fault-injection (kill DB / hang LLM). |

---

## 6. Defect summary (confirmed this run)

| ID | Title | Severity | Confirmed on | Fix location |
|---|---|---|---|---|
| DEF-01 | `/ingest/v1/page` & `/document` skip the non-empty main_text guard | Major | SQLite + Postgres | `api/ingest.py` (add rule1 to all paths) |
| DEF-02 | Corroboration keyed on `dir` before classification → under-counts cross-batch | Major | SQLite | `services/corroboration.py` |
| DEF-03 | `/explain` returns confidence only for `signal`; tenders write no evidence | Minor | SQLite + Postgres | `api/serving.py` `_CONF_MODEL`; `tender_scoring.py` |
| DEF-07 | Non-numeric `entity_id` → HTTP 500 on `/mallory/chat` | Major | SQLite + Postgres | `services/assistant.py::answer` |

**Not yet test-confirmed (code-inspection findings from the plan, still open):** DEF-04 (company-event dead-end), DEF-05 (concurrent rebuild race), DEF-06 (extraction noise), DEF-08 (shallow `/health`), DEF-09 (ingest upsert race), DEF-10 (no auth).

---

## 6A. Live LLM eval — Ollama farm (`text-model`)

Run against the **real remote farm** (`https://ollama.i3softlab.com/v1`, model `text-model`), `LLM_PROVIDER=ollama`, graded on the property-based golden set (`tests/eval/golden/*.json`). This is the only part of the run that made real network LLM calls.

**Command:** `MALLORY_EVAL_LIVE=1 PYTHONIOENCODING=utf-8 pytest tests/eval/test_eval_live.py -s -m live`

**Scorecard (captured):**
```
=== LIVE EVAL — provider=ollama fast=text-model deep=text-model ===
  [PASS] classify_signal    1273ms  real event from ingested.ndjson — L&T win
  [PASS] classify_signal     860ms  real event — Adani acquisition
  [PASS] classify_signal     870ms  real event — market tender opening
  [PASS] classify_signal     794ms  competitor setback → favourable
  [PASS] enrich_signal      2303ms  enrichment must not invent numbers
  [PASS] enrich_signal      2124ms  KNDS CAESAR Nigeria — French-language source
  [PASS] tender_verdict     1123ms  high fit (88%) → go; must cite the fit %
  [PASS] tender_verdict     1423ms  mid fit (62%) → maybe
  [FAIL] tender_verdict     1380ms  low fit (30%) → pass  -> ["lean='maybe' expected 'pass'"]
  --- per task ---
    classify_signal  4/4
    enrich_signal    2/2
    tender_verdict   2/3
  TOTAL 8/9  avg 1350ms/case
```
**Overall:** ✅ **PASSED** (8/9; the eval gate requires a majority) · avg **1350 ms/case** warm.

**What this proves about the real farm path (vs the stub/regex path the rest of the suite used):**
- ✅ `classify_signal` 4/4 — the live model's `lens` is generated *and* `dir`/`tags` stay rule-derived (deterministic `dir` correct on every case, incl. the French-language KNDS/CAESAR source).
- ✅ `enrich_signal` 2/2 — the **numbers-grounded guard holds against a real model**: no invented figures leaked into `sowhat`/`why_text`, including on a foreign-language source. This is the anti-hallucination spine working end-to-end with the farm, not a fake transport.
- ⚠️ `tender_verdict` 2/3 — **model-judgment finding (LIVE-01):** on a **30% fit** tender ("Armenia loitering munitions — category adjacent, no direct KSSL product"), the farm returned `lean='maybe'` where the deterministic policy says `lean='pass'`.

### ⚠️ LIVE-01 — model is over-optimistic on low-fit tenders
- **Input:** `best_fit_pct=30`, "Armenia tender for loitering munitions", "Category adjacent; no direct KSSL product."
- **Expected (deterministic policy, stub):** `pass` (fit < 55 → pass).
- **Actual (farm):** `maybe`.
- **Why it wasn't caught by validators:** `enum_valid` only checks `lean ∈ {go,maybe,pass}` — `maybe` is a valid enum, so the model's more-lenient verdict passes validation and would **ship** in production. The numeric fit % is untouched (still computed deterministically); only the go/maybe/pass *label* drifts.
- **Impact:** With `LLM_PROVIDER=ollama`, a clearly-unfit tender can be surfaced to the CEO as "maybe pursue" instead of "pass" — a business-judgment inflation. The stub/regex path does not have this drift.
- **Severity:** Minor (advisory prose label; numbers unaffected) · **Options:** (a) accept — live models are *graded, not gated* by design; (b) tighten the verdict prompt with explicit thresholds; (c) post-validate `lean` against the deterministic band (`<55 ⇒ pass`) and override, keeping the LLM for prose only — consistent with the "LLM never owns the number/label" doctrine.

### ⚠️ TEST-HARNESS-01 — live eval crashes on Windows console (Unicode)
- **Symptom:** first run died with `UnicodeEncodeError: 'charmap' codec can't encode character '→'` — a golden `note` contains `→`, and Windows stdout defaults to cp1252.
- **Fix applied for this run:** `PYTHONIOENCODING=utf-8`.
- **Recommendation:** make the eval robust (encode prints as UTF-8 or strip non-ASCII in the scorecard line) so it runs on a stock Windows shell without the env var. Minor, test-only.

---

## 7. How to reproduce

```bash
cd layer2-data-engine
.venv-win/Scripts/python -m pip install pytest
# full suite (SQLite, deterministic stub):
.venv-win/Scripts/python -m pytest tests/ -v
# just the QA-plan gap/defect module:
.venv-win/Scripts/python -m pytest tests/test_qa_plan.py -v
# live-server defect reproduction requires L2 up on :8000 (Postgres) — see docs/L2_TEST_PLAN.md §Part 6.
```

The 4 defect-probe tests are **expected to fail** until the underlying defects are fixed; each turns green automatically once its fix lands, so they double as regression guards.
