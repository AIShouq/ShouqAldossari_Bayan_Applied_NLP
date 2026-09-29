# DATA CARD — Bayan

## Dataset identity

- Name/version: Bayan public course sample fixtures, repository snapshot used on 2026-09-29
- Source/creator: public Bayan Applied NLP course repository
- Source: https://github.com/almiyead-rgb/bayan-applied-nlp-course
- License/permission: follow the upstream course/data terms; this repository does not claim ownership of the course fixtures
- Intended educational tasks: preprocessing, topic/sentiment classification, NER, extractive QA, Arabic profiling, semantic search, evaluation, and serving smoke tests

## Composition

### Classification fixture

| Split | Rows | Notes |
|---|---:|---|
| train | 24 | predefined grouped split |
| validation | 8 | model selection/calibration |
| frozen public test | 8 | opened after model choices |
| total | 40 | Arabic and English examples |

The split validator reported **20 groups**, four topic labels, and **0 group overlap**.

### Other public fixtures used

- NER: **12** records
- QA: **10** records
- Arabic-profile fixture: **20** records
- semantic-search cases: **24** records
- semantic-search queries: **18** records
- course evaluation predictions fixture: **36** records

## Fields and labels

The classification fixture includes identifiers, text, language, topic, sentiment, group ID, and predefined split. Topic labels are `digital_service`, `health`, `permit`, and `transport`. Other files contain task-specific BIO/entity annotations, QA question/context/answers, Arabic variant labels, case summaries/resolutions, and retrieval relevance IDs.

## Collection/generation

The project uses the course's public/synthetic educational fixtures. No real beneficiary records were collected for this submission.

## Cleaning and preprocessing

- Display copy: preserve a safe display-oriented version after masking.
- PII masking: tested regex/canary masking before downstream model use.
- Preprocessing version: `shouq-bayan-preprocess-v1.0`.
- Arabic search profile: CAMeL Tools-based normalisation with golden tests.
- Max token length used by task models: **32** based on public-fixture token-length evidence.
- Observed truncation rate on the measured public sample: **0.0**.

## Split and leakage controls

- Use the provided split field rather than re-splitting the classification fixture.
- Group overlap check: **0**.
- Model selection is based on validation evidence.
- The frozen classification test is opened only after model choice.
- QA test examples are excluded from training and threshold tuning.

## Known gaps and risks

- Dialects/Arabizi: limited sample coverage; Gulf errors were visible in the evaluation exercise.
- Class balance: tiny public splits can distort Macro-F1 and accuracy.
- Synthetic-to-real gap: the course fixtures do not represent full production traffic.
- Annotation ambiguity: possible in NER/QA and sentiment on small samples.
- Small slices: project test language slices have only four examples each.
- Privacy: masking tested canaries is not equivalent to production-grade de-identification.

## Permitted and prohibited use

- Permitted: educational experimentation, reproducibility, debugging, and course evaluation.
- Prohibited/high-risk: using this sample system as an automated authority for real public-service decisions or processing private citizen records without a new privacy/security/validation process.
- Human review: required for any real-world interpretation or action.

## Maintenance

- Owner through GitHub: `AIShouq`
- Repository: https://github.com/AIShouq/ShouqAldossari_Bayan_Applied_NLP
- Any data change should trigger a new version note, leakage check, metric rerun, and search-index/model rebuild as appropriate.
