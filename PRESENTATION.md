# PRESENTATION — Bayan | عرض بيان

**Learner:** Shouq Khalid Aldossari  
**GitHub username:** AIShouq

## 1. Problem and user

Bayan is an educational bilingual Arabic/English NLP pipeline for public-service-style feedback. It accepts Arabic or English text, protects/masks tested sensitive patterns, and returns structured NLP outputs. It is not a production government decision system.

## 2. Architecture

Explain this flow from `README.md`: preprocessing/privacy → topic/sentiment + NER + QA + multilingual retrieval/reranking → unified response → evaluation/serving.

Key decisions to explain if asked:
- grouped split with no group overlap;
- validation-only QA/search threshold tuning;
- L2-normalised multilingual embeddings + FAISS + cross-encoder reranking;
- ONNX FP32 adoption after parity/performance comparison.

## 3. Demonstration

### Arabic example
Input: `أحتاج معرفة حالة طلب التصريح`

Saved output highlights:
- language: `ar`
- topic: `permit`
- sentiment: `neutral`
- entity: `طلب التصريح` → `SERVICE`
- similar case: `AR-010`, score ≈ **0.5466**
- response latency in saved demo: ≈ **165.15 ms**

### English example
Input: `The bus did not arrive on time`

Saved output highlights:
- language: `en`
- topic: `transport`
- sentiment: `negative`
- similar case: `EN-004`, score ≈ **0.6964**
- response latency in saved demo: ≈ **164.96 ms**

### Rejected input
Whitespace-only text → HTTP **422** with `text must not be empty`.

Full saved evidence: `reports/presentation_demo.json`.

## 4. Measured evidence

Quality examples:
- topic frozen-test Macro-F1: **0.8667**
- NER public-test entity F1: **0.6667**
- retrieval MRR@10: **0.7500 → 0.8194** after reranking

Performance evidence:
- PyTorch FP32 p95: **1027.27 ms**
- ONNX FP32 p95: **246.86 ms**
- ONNX FP32 prediction agreement: **1.0**
- final optimisation decision: `ADOPT_ONNX_FP32`

Limitation to state clearly: QA answerable EM/F1 were **0/0** on the tiny validation set, and sentiment did not beat its TF-IDF baseline on the tiny frozen public test.

## 5. Decision and ownership

Measured extension: dialect router.
- baseline accuracy: **0.5000**
- extension accuracy: **0.7778**
- benefit: **+0.2778**
- added p95 latency: **0.0876 ms**

One decision I can explain live: why ONNX FP32 was adopted while dynamic INT8 was rejected despite being smaller/faster—the INT8 quality tax was too high.

The talk is five minutes plus two minutes of individual verification, with up to five slides or equivalent.
