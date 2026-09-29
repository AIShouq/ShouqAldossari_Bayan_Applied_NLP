# Bayan Model Card | بطاقة نموذج بيان

## Project owner

- Learner: **Shouq Khalid Aldossari**
- Repository: https://github.com/AIShouq/ShouqAldossari_Bayan_Applied_NLP
- Evidence date: **2026-09-29**
- Final commit SHA: `PENDING_FINAL_FREEZE`

## Artefact 1 — Topic classification

- Base checkpoint: `distilbert/distilbert-base-multilingual-cased`
- Task: bilingual public-service topic classification
- Training mode: partial fine-tuning (last transformer block + task head)
- Labels: `digital_service`, `health`, `permit`, `transport`
- Preprocessing: `shouq-bayan-preprocess-v1.0`
- Grouped split: 24 train / 8 validation / 8 frozen public test; 0 group overlap
- Frozen-test Macro-F1: **0.8667** (n=8)
- Frozen-test accuracy: **0.8750**
- TF-IDF baseline Macro-F1: **0.7333**

## Artefact 2 — Sentiment classification

- Base checkpoint: `distilbert/distilbert-base-multilingual-cased`
- Task: sentiment classification
- Frozen-test Macro-F1: **0.5333** (n=8)
- Frozen-test accuracy: **0.7500**
- TF-IDF baseline Macro-F1: **0.6667**

**Known limitation:** the transformer did not beat the baseline on the tiny public frozen test.

## Artefact 3 — NER

- Base checkpoint: `distilbert/distilbert-base-multilingual-cased`
- Task: BIO token classification with subword alignment and constrained BIO decoding
- Validation strict entity F1: **0.8571**
- Public-test strict entity F1: **0.6667**
- Public-test precision/recall: **1.0000 / 0.5000**

## Artefact 4 — Extractive QA

- Base checkpoint: `distilbert/distilbert-base-multilingual-cased`
- Task: extractive QA with no-answer decision
- Best fine-tune epoch: **8**
- Validation answerable EM/token F1: **0.0 / 0.0**
- Derived validation no-answer accuracy: **1.0** (n=2)
- Public-test no-answer accuracy: **0.5** (n=2)
- Null threshold calibrated on validation only: **-2.9909**

**Known limitation:** answer span extraction is weak in this run and should not be presented as production-ready QA.

## Artefact 5 — Semantic retrieval and reranking

- Sentence encoder: `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2`
- Index: FAISS `IndexFlatIP`
- Vector contract: L2-normalised sentence vectors
- Reranker: `cross-encoder/mmarco-mMiniLMv2-L12-H384-v1`
- Before rerank: Recall@10 **1.0000**, MRR@10 **0.7500**
- After rerank: Recall@10 **1.0000**, MRR@10 **0.8194**
- Cross-lingual slice (n=4): Recall@10 **1.0000**, MRR@10 **0.7500**

## Intended use

Educational demonstration of bilingual Arabic/English NLP for public-service-style feedback on synthetic/public course data.

## Out of scope

- real beneficiary or citizen profiling;
- automated government decisions;
- production deployment without additional validation, security, privacy, fairness, and monitoring work;
- claims of broad Arabic dialect coverage.

## Behavioural checks

| Capability | Result |
|---|---:|
| invariance | 2/2 PASS |
| minimum functionality | 4/4 PASS |
| empty-input rejection | HTTP 422 PASS |
| unified API tests | PASS |

## Limitations and risks

1. Very small public evaluation sets create high uncertainty.
2. QA answer extraction is poor in the executed run.
3. Dialect, Arabizi, and long-input coverage are incomplete.
4. Sentiment performance is weaker than its simple baseline on the frozen public test.
5. Metrics are `MEASURED_SMOKE`, not official production or cohort acceptance results.

## Ethical and privacy notes

Only public/synthetic course fixtures are used. The preprocessing layer masks tested PII patterns, but it is not a complete production de-identification solution. Human review is required before any real-world action.

## Reproduction

Run `Bayan_Capstone_Shouq_Aldossari.ipynb` from a clean Google Colab runtime and compare the resulting evidence with `EVALUATION_REPORT.md`, `BENCHMARKS.md`, and `reports/technical_summary.json`.
