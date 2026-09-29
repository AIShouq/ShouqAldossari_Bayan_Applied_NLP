# Bayan Evaluation Report | تقرير تقييم بيان

## 1. Scope

- Run date: **2026-09-29**
- Final commit SHA: `PENDING_FINAL_FREEZE`
- Runtime/device: Google Colab; CPU model-only optimisation comparison plus Colab HTTP serving smoke
- Data: public/synthetic Bayan course fixtures
- Preprocessing: `shouq-bayan-preprocess-v1.0`; Arabic search profile with CAMeL Tools
- Main encoder/checkpoint: `distilbert/distilbert-base-multilingual-cased`
- Search encoder: `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2`
- Reranker: `cross-encoder/mmarco-mMiniLMv2-L12-H384-v1`
- Result label: **MEASURED_SMOKE**

## 2. Contracts before measurement

| Contract | Evidence | Status |
|---|---|---|
| no real PII | public/synthetic fixtures; masking canaries | PASS |
| grouped split without overlap | 24/8/8; group overlap = 0 | PASS |
| tokenizer/model aligned | same multilingual DistilBERT checkpoint family in task pipeline | PASS |
| Arabic profile documented | versioned preprocessing + CAMeL golden tests | PASS |
| retrieval vectors L2 normalised | notebook assertion before FAISS indexing | PASS |
| frozen test not used for tuning | classification test opened after validation model choice; QA test excluded from training/tuning | PASS |

## 3. Task results

| Task | Metric | Result | Evaluation set |
|---|---|---:|---|
| Topic classification | Macro-F1 | **0.8667** | frozen public test, n=8 |
| Topic baseline | Macro-F1 | 0.7333 | same test |
| Sentiment classification | Macro-F1 | **0.5333** | frozen public test, n=8 |
| Sentiment baseline | Macro-F1 | 0.6667 | same test |
| NER | strict entity F1 | **0.6667** | public test; 4 true entities |
| NER validation | strict entity F1 | 0.8571 | validation; 4 true entities |
| QA answerable | EM / token F1 | **0.0000 / 0.0000** | official answerable validation, n=2 |
| QA derived no-answer | accuracy | **1.0000** | derived validation, n=2 |
| QA public no-answer | accuracy | **0.5000** | public test, n=2 |
| Retrieval before rerank | Recall@10 / MRR@10 | **1.0000 / 0.7500** | answerable public queries, n=12 |
| Retrieval after rerank | Recall@10 / MRR@10 | **1.0000 / 0.8194** | same queries |

## 4. Slices and uncertainty

### Project classifier language slices

| Task | Slice | n | Macro-F1 |
|---|---|---:|---:|
| topic | Arabic | 4 | 0.6667 |
| topic | English | 4 | 1.0000 |
| sentiment | Arabic | 4 | 0.7333 |
| sentiment | English | 4 | 0.6000 |

### Course evaluation fixture

The course prediction fixture (36 rows) was used for slice/bootstrap/error-analysis mechanics. It is not presented as the trained project classifier's final test output.

- prediction_b Gulf slice: n=12, Macro-F1 **0.6583**
- bootstrap 95% CI for that Gulf estimate: approximately **[0.3000, 0.9031]**
- prediction_b long-input slice: n=18, Macro-F1 **0.5259**
- prediction_b short-input slice: n=18, Macro-F1 **0.8286**

The wide Gulf interval and tiny project test slices are a warning against strong generalisation claims.

## 5. Version comparison

For semantic retrieval:
- Model A: multilingual embedding retrieval before reranking
- Model B: same retrieval pipeline plus cross-encoder reranking
- Recall@10: **1.0000 → 1.0000**
- MRR@10: **0.7500 → 0.8194**

The observed MRR improvement supports keeping reranking for this public smoke workload, while the small sample prevents a broad statistical claim.

## 6. Behavioural tests

| Type | passed/total | pass rate | Note |
|---|---:|---:|---|
| invariance | 2/2 | 100% | tested notebook canaries |
| minimum functionality | 4/4 | 100% | Arabic/English topic examples |

A separate empty-input API case returned HTTP **422** with `text must not be empty`.

## 7. Error analysis

Only **8** public fixture errors were available for the notebook taxonomy exercise; therefore this does **not** meet a hypothetical ≥100-error manual-review standard and should not be described as such.

| Taxonomy tag | Count | Priority response |
|---|---:|---|
| dialect_gap | 4 | measure dialect-aware routing/profile comparison |
| class_confusion | 2 | inspect hard examples and per-class errors |
| truncation | 2 | re-audit long-input/max-length behaviour |

## 8. Prioritised fixes

1. **Gulf/dialect gap:** compare measured dialect-aware routing/profile behaviour and verify on Gulf slice + CI.
2. **Long-input risk:** re-audit length distribution and compare longer max-length settings on long slices.
3. **Topic class confusion:** review class-specific failures and add targeted behavioural cases.

## 9. Known limitations

- The public fixtures are small and intended for course smoke evidence.
- Sentiment underperformed its TF-IDF baseline on the tiny frozen public test.
- QA answerable validation EM/F1 were 0 in this run; QA is the largest quality weakness.
- Dialect, Arabizi, and long-input coverage remain limited.
- Colab timing is noisy and not a reference production environment.
- No production/government decision claim is made.

## 10. Management summary

Bayan successfully demonstrates the required end-to-end technical workflow: protected/versioned preprocessing, bilingual task models, entity extraction, semantic retrieval with measured reranking, slice/error analysis, optimisation comparison, unified API tests, and a measured extension. Topic classification and retrieval produced the strongest public-fixture evidence; NER is partially successful; sentiment is mixed; QA answer extraction remains weak.

The next evidence-driven work should focus on QA data/training, broader dialect and long-input evaluation, and larger frozen evaluation cohorts. The current evidence supports an educational prototype and engineering demonstration, not production readiness.
