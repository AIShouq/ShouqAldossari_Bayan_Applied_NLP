# BENCHMARKS — Bayan

## 1. Claim boundary

- Artefact role: `PROJECT_ARTIFACT`
- Result label: `MEASURED_SMOKE`
- Task measured for optimisation: topic-classification project artefact
- Decision date: 2026-09-29
- Author: Shouq Khalid Aldossari

The measurements below are from the executed public-fixture Colab run. They are project smoke evidence, not official reference-device acceptance.

## 2. Performance objective

The engineering objective was to reduce model-only latency and/or improve throughput without changing FP32 predictions. No course reference-device SLA was claimed before these measurements; therefore this report does **not** retroactively invent a numeric pass/fail budget.

## 3. Reproduction contract

| Field | Value |
|---|---|
| Runtime | Google Colab, course reference environment |
| Python | 3.12.13 course reference |
| Device/provider | CPU model-only comparison; HTTP serving in Colab runtime |
| Base model | `distilbert/distilbert-base-multilingual-cased` |
| Preprocessing version | `shouq-bayan-preprocess-v1.0` |
| Workload | public Bayan topic-classification project workload |
| Batch/items per call | 8 |
| Max length | 32 |
| Warm-up | 3 calls |
| Measured repetitions | 30 per model-only candidate |
| Boundary | model-only for candidate comparison; separate HTTP end-to-end smoke |
| Memory method | observed process RSS |

## 4. Controlled candidates

| ID | Runtime/precision | Only intended change | Size MiB |
|---|---|---|---:|
| A | PyTorch FP32 reference | baseline runtime | 519.03 saved project model |
| B | ONNX Runtime FP32 | export/runtime | 516.36 |
| C | ONNX Runtime dynamic INT8 | dynamic weight quantisation | 129.46 |

## 5. Parity

| Comparison | max abs logits diff | prediction agreement | Verdict |
|---|---:|---:|---|
| A vs B | 2.62e-06 | 1.0000 | PASS |
| A vs C | not used as adoption parity evidence | quality tax measured separately | REJECT FOR ADOPTION |

FP32 parity is the basis for the selected ONNX FP32 serving path.

## 6. Model-only performance results

| ID | p50 ms | p95 ms | p99 ms | items/s | observed peak RSS MiB |
|---|---:|---:|---:|---:|---:|
| A — PyTorch FP32 | 211.09 | 1027.27 | 1375.41 | 20.12 | 3744.43 |
| B — ONNX FP32 | 161.58 | 246.86 | 252.26 | 45.42 | 4505.70 |
| C — ONNX INT8 | 107.74 | 119.40 | 122.34 | 73.40 | 4404.82 |

Approximate throughput speedup vs A:
- B: **2.26×**
- C: **3.65×**

## 7. Quality decision

- FP32 ONNX prediction agreement with PyTorch: **1.0**
- FP32 ONNX quality tax on prediction agreement: **0**
- Dynamic INT8 measured quality tax: **0.5655**

The INT8 candidate is faster and smaller, but the measured quality degradation is too large for adoption in this submission.

## 8. HTTP serving smoke

Serving engine: `onnxruntime_fp32`

| Metric | Result |
|---|---:|
| concurrency | 16 |
| warm-up requests | 16 |
| measured requests | 64 |
| p50 | 1017.36 ms |
| p95 | 1182.84 ms |
| p99 | 1454.39 ms |
| throughput | 15.48 requests/s |

These HTTP numbers are environment-specific Colab smoke measurements and are not an official production SLA.

## 9. Decision

- Selected runtime: **ONNX Runtime FP32**
- Decision: **ADOPT_ONNX_FP32**
- Reason: materially better model-only p95 and throughput with 100% prediction agreement on the benchmark workload.
- Known limitation: small public workload and Colab timing noise.
- Rollback: use the original PyTorch FP32 saved model if future parity or compatibility checks fail.

## 10. Reproduction

Run `Bayan_Capstone_Shouq_Aldossari.ipynb` from a clean Colab runtime and execute the optimisation/serving section as part of **Run all**. Do not commit generated weights or ONNX artefacts.
