# Industry Problem Brief
**Project:** Low-Resource Indian Language Intelligence with Parameter-Efficient Adaptation
**Author:** <your name> | **Date:** 08-10-2026 | **Version:** 0.1

## 1. Problem Statement
Public-service teams and language-technology researchers need to analyse citizen text (comments, feedback, social posts) written in Hindi, code-mixed Hinglish and other Indian languages. Labelled data is scarce, English-trained models perform unevenly across scripts and dialects, and annotation is expensive. This project builds a prototype that curates and normalizes such text, adapts a multilingual model with parameter-efficient fine-tuning (LoRA), uses active learning to cut labelling cost, and reports performance, calibration and per-language disparity transparently.

## 2. Stakeholder Map
| Stakeholder | Role | Interest | Influence |
|---|---|---|---|
| Public-service analyst | Primary user | Reliable sentiment signals from citizen text, with confidence | High |
| Language-technology researcher | Primary user | Reproducible experiments, per-language results | High |
| Annotator | Secondary user | Efficient labelling queue, clear guidelines | Medium |
| Administrator / ML engineer | Operator | Model governance, monitoring, deployment | High |
| Citizens (data subjects) | Affected party | Privacy, fair treatment across languages and dialects | Low (but critical) |
| Evaluator / auditor | Reviewer | Evidence, audit trail, reproducibility | High |

## 3. Current Process (Pain Points)
- Analysts read comments manually or use English-only tools that misread Hinglish and Roman-script Hindi.
- Labels are scarce, so models are trained on small noisy sets with no uncertainty estimates.
- Results are reported as one overall accuracy, hiding large gaps between languages.
- Experiments are not reproducible: no fixed splits, no tracking, no model documentation.

*Evidence to add:* 2-3 cited sources (papers or reports) on code-mixing difficulty and low-resource Indian NLP.

## 4. Personas and User Stories
**Persona A: Asha, public-service analyst.** Reviews thousands of citizen comments weekly, not a machine learning expert.
**Persona B: Dr. Rao, NLP researcher.** Wants to compare adaptation methods fairly.
**Persona C: Imran, administrator.** Responsible for safe deployment.

| ID | User story | Acceptance criteria |
|---|---|---|
| US1 | As an analyst, I submit text and get a sentiment label with a confidence score. | API returns label, calibrated probability and detected language in under 1 s on CPU. |
| US2 | As an analyst, I see when the model is unsure so I can route the item to review. | Items below a confidence threshold are flagged "needs review". |
| US3 | As an annotator, I get the most informative unlabeled samples first. | Active-learning queue ranks samples by uncertainty; selection is logged. |
| US4 | As a researcher, I rerun an experiment from a config file and get the same results. | Fixed seeds, versioned splits, config stored with each run. |
| US5 | As a researcher, I see macro F1 per language with confidence intervals. | Report shows bootstrap 95% CI per language. |
| US6 | As an administrator, I see which model version produced each prediction. | Every prediction is written to an audit log with model version and timestamp. |
| US7 | As an administrator, I get alerted when input data drifts. | Drift report generated on recent inputs vs. training data. |

## 5. Misuse and Abuse Cases
| ID | Misuse | Mitigation |
|---|---|---|
| M1 | Using the system to profile or surveil individual citizens | No user identifiers stored; text de-identified; scope limited to aggregate analysis; documented in model card |
| M2 | Biased moderation against a dialect or script (e.g. Roman Hindi flagged more often) | Per-language and per-script disparity reporting with go/no-go gap threshold |
| M3 | Data poisoning through malicious labels | Annotator roles, label audit trail, agreement checks on a gold subset |
| M4 | Malicious or oversized API input (injection, huge payloads) | Pydantic validation, length limits, rate limiting, safe logging |
| M5 | Overtrust in low-confidence predictions | Calibrated scores, "needs review" band, documented limitations |
| M6 | Leaking personal data through logs or the repository | De-identification step, `.gitignore` for raw data and secrets, no PII in logs |

## 6. Scope
**In scope:** Sentiment classification for Hindi, Hinglish and one further Indian language; normalization and transliteration; baseline vs. LoRA comparison; active learning; evaluation (macro F1, ECE, annotation efficiency, language gap, robustness); FastAPI service; monitoring; model card.

**Out of scope:** Real-time social media scraping; individual-level user tracking; training a model from scratch; production Kubernetes cluster (documented as roadmap only); languages beyond the chosen three.

## 7. Non-Functional Requirements
- **Reproducibility:** fixed seeds, versioned splits, experiment config logged in MLflow.
- **Security and privacy:** no secrets or personal data in the repo; input validation; role-based access for annotator vs. admin endpoints.
- **Performance:** CPU inference under 1 s per request for the baseline.
- **Observability:** structured logs, request metrics, drift report.
- **Portability:** runs from README steps or a Docker image on a clean machine.

## 8. Risks
| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Suitable public dataset unavailable or licence unclear | Medium | High | Verify licence early; keep a fallback dataset; record provenance |
| Noisy labels cap performance | High | Medium | Report label-noise caveat; analyse errors |
| No local GPU (GT 610) | Certain | Medium | Train on Colab/Kaggle; baseline and serving on CPU |
| One-day timeline | Certain | High | MoSCoW scoping; core pipeline first |
| Train/test leakage from duplicates | Medium | High | Deduplication and leakage check before splitting |

## 9. Success Metrics and Go/No-Go Criteria
| Metric | Target (go) | No-go |
|---|---|---|
| Macro F1 (overall) | LoRA model beats TF-IDF baseline by a measurable margin with non-overlapping or reported 95% CI | No improvement over baseline |
| Calibration (ECE) | ECE <= 0.10 after temperature scaling | ECE > 0.15 |
| Annotation efficiency | Uncertainty sampling reaches target F1 with fewer labels than random sampling | No advantage over random |
| Language disparity | Macro F1 gap between best and worst language <= 0.10 (report if exceeded) | Gap unreported |
| Robustness | Macro F1 drop <= 0.10 under typo/script-mix perturbation | Drop > 0.15 |
| Engineering | Tests pass, container builds, README setup works on a clean machine | Setup fails |

*Thresholds are initial and pre-registered; adjust only before running final experiments and document any change.*

## 10. Prioritized Backlog (MoSCoW)
**Must:** data curation with provenance and leak-free splits; normalization and transliteration; TF-IDF baseline; LoRA multilingual model; evaluation (macro F1, ECE, per-language); active-learning experiment; robustness test; FastAPI with validation and audit log; README; model card.
**Should:** SHAP explanations and error analysis; MLflow tracking; Dockerfile; unit and integration tests; drift report; simple error dashboard.
**Could:** GitHub Actions CI; dependency scan; Optuna tuning; DVC for data versioning.
**Won't (this release):** Kubernetes deployment; Airflow/Prefect orchestration; additional languages.
