# Post-POC Model Upgrade Menu (Production Options)

| Field | Value |
|---|---|
| Document | Companion to the POC PRD v1.1 |
| Status | Reference / decision menu — **not** a POC requirement |
| Version | 1.0 — June 2026 |

## How to read this document

The **POC PRD v1.1 is unchanged**: during the POC, every model is open-weight and self-hosted (Ollama + open runtimes), zero paid services. That constraint stays in force until the architecture is proven on your golden set.

This document is the **"what we can swap to later"** menu. Because the POC's Model Gateway is OpenAI-compatible and every model choice is pinned in `pipeline.yaml` behind a role (not hardcoded), each row below is a **drop-in swap at one config line** once you decide a given component needs more quality. The architecture does not change — only the model behind a role does.

**Per row:** Role · POC default (open) · Production upgrade options (open *and* paid) · When to upgrade · Trade-off · Data-sovereignty note.

> **Sovereignty flag.** Rows marked 🔒 must stay self-hosted if the corpus is sensitive — sending that content to a hosted API may be unacceptable regardless of quality. Rows marked 🌐 are safer to send out (e.g. embedding a public-web subset) but still need a data-policy sign-off. Decide this **per corpus subset**, not globally.

---

## 1. Generation & reasoning (the biggest quality lever)

| | |
|---|---|
| **Role** | Synthesis / planner / final answer |
| **POC default** | Qwen3-32B (or Llama-3.3-70B) via Ollama |
| **Open upgrades** | Llama-3.3-70B, Qwen3-72B, DeepSeek-V3 / R1-class, Mistral-Large-class — served via **vLLM/TGI/SGLang** (higher batched throughput than Ollama) |
| **Paid upgrades** 🔒 | Claude (Opus/Sonnet tier), GPT-class, Gemini Pro — via API |
| **When** | Golden-set shows synthesis errors on multi-hop/global (QH/QG) that bigger open models don't fix; or you need 200k–1M context for whole-document reasoning |
| **Trade-off** | Paid frontier ≈ best reasoning + long context, but recurring per-token cost + data leaves boundary; large open models close much of the gap at fixed hardware cost |

**Workhorse generation (Fast mode):** POC = Qwen3-8B. Upgrades: Qwen3-14B/32B open, or a cheap fast hosted tier (Haiku/Flash-class) 🔒 if latency-bound and data permits.

**SLM roles (router, CRAG grader, citation-aligner):** POC = Qwen3-1.7B/4B. These should **stay small and open even in production** — they run on every query and frontier models here just burn money for no quality gain. Upgrade path is *fine-tuning* the open SLM (Phase 6), not swapping to a bigger model.

---

## 2. Embeddings

| | |
|---|---|
| **Role** | Text dense embeddings |
| **POC default** | BGE-M3 (Ollama) |
| **Open upgrades** | gte-Qwen2-7B-instruct, NV-Embed-class, E5-mistral — served via TEI/vLLM |
| **Paid upgrades** 🌐 | Voyage-3-class, Cohere Embed-4, OpenAI text-embedding-3-large |
| **When** | Retrieval recall plateaus and ablation shows the embedder (not chunking/fusion) is the bottleneck |
| **Trade-off** | 7B open embedders cost 5–10× compute vs BGE-M3; paid embedders are strong + zero ops but re-embedding the whole corpus on an API meters cost and sends content out. **Re-embedding = full corpus pass; version embeddings so you can A/B.** |

**Multimodal/image embeddings:** POC = jina-clip-v2 / SigLIP-2 open. Paid upgrade 🌐: Cohere Embed-4, voyage-multimodal-3.x (unified text+image space, strong, but hosted).

---

## 3. Reranker

| | |
|---|---|
| **POC default** | BGE-reranker-v2-m3 / Jina-reranker-v2 (open runtime) |
| **Open upgrades** | Jina-reranker-v3 (listwise), larger BGE variants, fine-tuned SLM grader |
| **Paid upgrades** 🌐 | Cohere Rerank 3.5 |
| **When** | Precision@k is the gap after retrieval recall is solid |
| **Trade-off** | Cohere Rerank is strong and zero-ops but per-call billing on every query (rerank runs on *every* request — this meters fast); open rerankers on a small GPU are usually the better long-run economics |

---

## 4. Visual document retrieval

| | |
|---|---|
| **POC default** | ColQwen2.5 (open runtime) |
| **Open upgrades** | ColQwen3-class, ColModernVBERT (10× smaller, near-parity), pooling + two-stage search for scale |
| **Paid upgrades** 🌐 | Managed document-intelligence APIs (Azure Doc Intelligence, etc.) — generally *more expensive* per page than self-hosting ColQwen, so this is rarely the upgrade you want |
| **When** | Visual recall plateaus or storage/latency forces a smaller model |
| **Trade-off** | Mostly an open-vs-open decision (size/speed/storage); paid here usually loses on cost |

---

## 5. ASR / OCR / parsing

| | |
|---|---|
| **ASR** POC = Whisper large-v3 (WhisperX). Open upgrades: NVIDIA Canary/Parakeet (faster). Paid 🔒: hosted ASR — rarely worth it for sensitive audio. |
| **OCR** POC = Surya/PaddleOCR + Qwen-VL fallback. Open is state-of-competitive; paid OCR usually loses on cost at corpus scale. |
| **Parsing** POC = Docling + PyMuPDF. Paid 🌐: LlamaParse, Reducto, Azure — strong on gnarly tables, but metered per page; consider only for a hard document subset, not the whole corpus. |
| **When** | Per-format parse-quality sampling (FR-ING-15) shows a specific format failing and no open tool fixes it |

---

## 6. Verification / NLI

| | |
|---|---|
| **POC default** | Qwen3-4B-as-judge + open NLI checker |
| **Open upgrades** | Larger open judge model; dedicated open NLI/AlignScore-class checker |
| **Paid upgrades** 🔒 | Frontier model as judge (strong, but the judge sees answer + evidence = sees corpus content) |
| **Rule** | Judge model MUST differ from generator (FR-VER-2). A frontier judge over open-generated answers is a reasonable late upgrade if data policy allows. |

---

## 7. Serving runtime (not a model — but the key enabler)

| | |
|---|---|
| **POC** | Ollama (simple, single-node, OpenAI-compatible) |
| **Production** | vLLM / TGI / SGLang (all open-source) — same open weights, much higher batched throughput; or a hosted inference provider 🔒 |
| **When** | Concurrent users exceed ~5–10 and Ollama throughput becomes the bottleneck (NFR-3 scaling) |
| **Why it's painless** | The Model Gateway already speaks OpenAI-compatible; swapping the backend is a deployment change, not an app-code change. **This is the single most important reason the POC won't trap you.** |

---

## 8. Decision rules (so upgrades stay disciplined)

1. **Never upgrade on vibes — upgrade on a measured gap.** Run the golden-set ablation (FR-EVAL-3): if swapping a model doesn't move the metric for the failing query class, don't pay for it.
2. **Upgrade one role at a time**, re-run eval, keep the config-hash trace. Multi-swap = un-diagnosable.
3. **Sovereignty before quality.** A paid model that gives +5% accuracy is worthless if that corpus subset can't legally leave the boundary. Resolve 🔒/🌐 per subset first.
4. **SLM roles stay open + small.** Per-query helpers (router/grader) should be fine-tuned, not enlarged or sent to APIs.
5. **Cost shape flips with paid APIs.** Self-hosted = fixed hardware cost, zero marginal/query. Paid = near-zero capital, metered per query — rerankers and embedders are the dangerous ones because they fire on *every* request.
6. **Production serving = vLLM/TGI on open weights** is often the highest-leverage upgrade: same models, more throughput, still sovereign, no per-token bill.

---

## 9. Quick-reference upgrade table

| Role | POC (open, Ollama/runtime) | Best open upgrade | Best paid option | Paid worth it? |
|---|---|---|---|---|
| Synthesis | Qwen3-32B | Llama-3.3-70B / Qwen3-72B / DeepSeek | Claude/GPT/Gemini top tier 🔒 | Sometimes — hardest reasoning + long context |
| Fast gen | Qwen3-8B | Qwen3-14B/32B | Haiku/Flash-class 🔒 | Rarely |
| SLM helpers | Qwen3-1.7B/4B | fine-tuned same | — | No |
| Text embed | BGE-M3 | gte-Qwen2-7B / NV-Embed | Voyage / Cohere / OpenAI 🌐 | Sometimes |
| MM embed | jina-clip-v2 | SigLIP-2 | Cohere Embed-4 / voyage-mm 🌐 | Sometimes |
| Reranker | BGE-reranker-v2 | Jina-reranker-v3 | Cohere Rerank 3.5 🌐 | Rarely (fires every query) |
| Visual retr. | ColQwen2.5 | ColQwen3 / ColModernVBERT | (managed doc-AI) 🌐 | No (cost) |
| ASR | Whisper large-v3 | Canary/Parakeet | hosted ASR 🔒 | No |
| OCR | Surya/PaddleOCR | Qwen-VL OCR | LlamaParse/Azure 🌐 | Subset only |
| Parsing | Docling/PyMuPDF | — | LlamaParse/Reducto 🌐 | Hard subset only |
| Verification | Qwen3-4B judge | larger open judge | frontier judge 🔒 | Sometimes |
| Serving | Ollama | **vLLM/TGI/SGLang** | hosted inference 🔒 | Open vLLM usually wins |

*🔒 = keep self-hosted if corpus sensitive · 🌐 = sendable with data-policy sign-off · All "best open upgrade" options remain zero-marginal-cost and on-prem.*
