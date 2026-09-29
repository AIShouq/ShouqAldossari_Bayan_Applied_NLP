# DECISIONS — Bayan

**Owner:** Shouq Khalid Aldossari  
**Evidence date:** 2026-09-29

## Decision D-001 — Keep versioned privacy-aware preprocessing

- **Gate:** A
- **Status:** accepted

### Context
The system needs a safe display/model-text separation and reproducible Arabic/English preprocessing without silently changing raw evidence.

### Decision
Use preprocessing version `shouq-bayan-preprocess-v1.0`, mask tested PII patterns, keep a display copy, and apply a documented Arabic search profile using CAMeL Tools for search-oriented normalisation.

### Evidence
- PII masking recall on notebook canaries: **1.0**
- Arabic golden tests: **PASS**
- Chosen max length: **32**
- Observed public-fixture truncation rate: **0.0**

### Consequences and rollback
The approach is reproducible and privacy-aware for the tested patterns, but it is not a complete production PII detector. Roll back any normalisation rule that changes meaning or breaks golden tests.

---

## Decision D-002 — Use grouped predefined splits and report baselines

- **Gate:** B
- **Status:** accepted

### Context
The classification fixture includes `group_id` and predefined train/validation/test splits. Leakage between related examples would invalidate the evaluation.

### Decision
Respect the supplied grouped split: **24 train / 8 validation / 8 frozen test**, with **0 group overlap**. Use character TF-IDF + LinearSVC as a baseline and multilingual DistilBERT task heads as the transformer comparison.

### Evidence
- Topic frozen-test Macro-F1: baseline **0.7333**, transformer **0.8667**
- Sentiment frozen-test Macro-F1: baseline **0.6667**, transformer **0.5333**

### Consequences and rollback
The topic transformer improved over baseline on the public smoke split; sentiment did not. Sentiment is therefore documented as a limitation rather than presented as an improvement.

---

## Decision D-003 — Use validation-only QA no-answer threshold calibration

- **Gate:** B
- **Status:** accepted

### Context
Extractive QA needs a no-answer path. Tuning on the frozen test would create leakage.

### Decision
Train with answerable training examples plus a small set of derived training no-answer examples, select the QA checkpoint using validation loss, and calibrate the null threshold on validation only. The frozen test is not used for training/tuning.

### Evidence
- Best QA fine-tune epoch: **8**
- Validation answerable EM/F1: **0.0 / 0.0**
- Derived validation no-answer accuracy: **1.0**
- Public test no-answer accuracy: **0.5** (n=2)

### Consequences and rollback
The no-answer mechanism works on the derived validation cases, but answer extraction remains weak. Do not claim strong QA quality; larger QA data and stronger span training are future work.

---

## Decision D-004 — Use multilingual embeddings + FAISS + measured reranking

- **Gate:** C
- **Status:** accepted

### Decision
Use `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2`, L2-normalised embeddings, FAISS `IndexFlatIP`, and `cross-encoder/mmarco-mMiniLMv2-L12-H384-v1` for reranking.

### Evidence
- Before rerank: Recall@10 **1.0000**, MRR@10 **0.7500**
- After rerank: Recall@10 **1.0000**, MRR@10 **0.8194**
- Cross-lingual slice (n=4): Recall@10 **1.0000**, MRR@10 **0.7500**
- Validation search no-answer threshold: **0.4804**, validation accuracy **1.0**

### Consequences and rollback
Reranking improved MRR on the public queries while keeping Recall@10. The evidence is small-sample smoke evidence only.

---

## Decision D-005 — Prioritise dialect gap, long inputs, then class confusion

- **Gate:** C
- **Status:** accepted

### Evidence
Error taxonomy on the available public fixture errors:
- dialect gap: **4**
- class confusion: **2**
- truncation: **2**

### Decision
Prioritise: (1) dialect-aware routing/profile comparison, (2) long-input/max-length re-audit, (3) class-specific confusion analysis.

---

## Decision D-006 — Adopt ONNX Runtime FP32; reject INT8 for this submission

- **Gate:** D
- **Status:** accepted

### Context
The project needs a serving candidate that improves speed without an unacceptable quality change.

### Decision
Adopt **ONNX Runtime FP32** for the optimised topic-classification serving path. Dynamic INT8 was attempted and measured but is not adopted because its measured quality tax was too high.

### Evidence
- PyTorch p95: **1027.27 ms**, throughput **20.12 items/s**
- ONNX FP32 p95: **246.86 ms**, throughput **45.42 items/s**
- FP32 prediction agreement: **1.0**
- max absolute logits difference: **2.62e-06**
- ONNX INT8 p95: **119.40 ms**, throughput **73.40 items/s**
- INT8 artefact size: **129.46 MiB** vs ONNX FP32 **516.36 MiB**
- measured INT8 quality tax: **0.5655**

### Consequences and rollback
ONNX FP32 is faster while preserving predictions on the benchmark workload. If parity breaks on a future workload, roll back to the saved PyTorch FP32 path.

---

## Decision D-007 — Keep measured dialect router as a bounded extension

- **Gate:** D/E
- **Status:** accepted

### Evidence
- Baseline accuracy: **0.5000**
- Extension accuracy: **0.7778**
- Benefit: **+0.2778**
- Added p95 cost: **0.0876 ms**

### Decision
Keep the dialect router as a measured educational extension. Do not describe it as production-grade dialect detection.
