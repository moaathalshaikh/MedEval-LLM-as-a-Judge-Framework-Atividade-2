# MedEval — Human-Aligned LLM Evaluation Platform
### Activity 2 | Group 5 | Tópicos Especiais em ES e SI I
**Federal University of Sergipe — Prof. Glauco Carneiro — May 2026**

---

## 📋 Overview

MedEval is a research platform that implements the **LLM-as-a-Judge** paradigm to evaluate Small Language Models (SLMs) on medical question-answering tasks. It stores the entire evaluation lifecycle in a PostgreSQL relational database, enabling full traceability and reproducibility.

> *"It is not enough to evaluate — it is necessary to know **who** evaluated, **when**, and under **what explanatory logic**."*

The platform was designed and developed by Moaath Almohammad Alshaikh, who built the full-stack system — React frontend, Node.js backend, and PostgreSQL integration — on top of the initial database schema established by Marcelo West.

> — Prof. Glauco Carneiro

---

## 🎬 Video Presentation

▶️ **[Watch the full presentation (10–20 min)](YOUR_VIDEO_LINK_HERE)**

---

## 👥 Team

| Name | Role |
|---|---|
| Moaath Almohammad Alshaikh | Full-stack development, Frontend (React), Database architecture & integration, System design, Spearman analysis, Prompt engineering, Database architecture |
| Tasneem Alshaher | SLM Models, Dataset management |
| Marcelo West | Database architecture |
| Clélio Xavier | Judge pipeline |
| Sérgio Santos | Human evaluation, Results analysis |
| Hernandison Bispo | Research Insights |

---

## 📊 Key Results

| Metric | Value |
|---|---|
| **Spearman ρ (Judge vs Human)** | **0.956** |
| p-value | 0.0000 |
| Paired evaluations (n) | 92 |
| MCQ Accuracy | 34.6% |
| Datasets | K-QA (201 open-ended) + USMLE (54 MCQ) |
| SLMs evaluated | 6 models |
| LLM Judge | DeepSeek-v4-flash |

---

## 🗂️ Repository Structure

```
Atividade_2/
├── README.md                        ← This file
├── backup/
│   ├── medeval-backup-2026-05-14.sql     ← Full PostgreSQL dump
│   └── medeval-backup-2026-05-14.sql.sha256  ← SHA-256 checksum
├── exports/
│   ├── medeval-research-2026-05-14.jsonl ← Research dataset (JSONL)
│   └── medeval-research-2026-05-14.csv   ← Research dataset (CSV)
├── scripts/
│   ├── spearman_analysis.py         ← Spearman correlation script
│   ├── import_responses.py          ← Import SLM responses to DB
│   └── run_judge.py                 ← LLM Judge evaluation pipeline
├── prompts/
│   ├── judge_medical_open.txt       ← Judge prompt for open-ended questions
│   ├── judge_medical_mcq.txt        ← Judge prompt for MCQ questions
│   └── reference_answer.txt        ← Reference answer generation prompt
├── sql/
│   └── schema.sql                   ← DDL — CREATE TABLE statements
├── tutorial/
│   └── restore_tutorial.pdf         ← Step-by-step restore guide
└── presentation/
    └── activity2_presentation.pdf   ← Slides (PDF)
```

---

## 🗄️ Database Restore

### Requirements
- PostgreSQL 14+
- `psql` CLI

### Steps

**1. Create the database:**
```bash
createdb -U postgres medeval_db
```

**2. (Optional) Verify backup integrity:**
```bash
sha256sum -c medeval-backup-2026-05-14.sql.sha256
```

**3. Restore:**
```bash
psql -U postgres -d medeval_db -f backup/medeval-backup-2026-05-14.sql
```

**4. Verify:**
```sql
SELECT COUNT(*) FROM questions_w_answers;   -- should return 255
SELECT COUNT(*) FROM judge_evaluations;     -- should return 92+
SELECT COUNT(*) FROM model_responses;
```

> Full instructions with screenshots are in `tutorial/restore_tutorial.pdf`.

---

## ⚙️ Evaluation Pipeline

```
Dataset (K-QA / USMLE)
    │
    ▼
SLM Response Generation  ←── 6 candidate models (Activity 1)
    │
    ▼
LLM Reference Answer     ←── DeepSeek-v4-flash generates synthetic reference
    │
    ▼
Judge Evaluation         ←── Score 1–5 + structured reasoning per response
    │
    ▼
Human Review             ←── Independent scores from team members
    │
    ▼
Spearman ρ               ←── Measures Judge vs Human agreement
    │
    ▼
Research Insights        ←── Bias detection, error taxonomy, disagreement analysis
```

---

## 🔬 Spearman Correlation

We use Spearman's rank correlation to measure how well the LLM Judge agrees with human reviewers:

$$\rho = 1 - \frac{6 \sum d_i^2}{n(n^2 - 1)}$$

Where $d_i$ is the difference between judge score and human score for each response.

```python
from scipy.stats import spearmanr

correlation, p_value = spearmanr(human_scores, judge_scores)
print(f"Spearman ρ = {correlation:.3f}, p = {p_value:.4f}")
# Output: Spearman ρ = 0.956, p = 0.0000
```

| ρ range | Interpretation |
|---|---|
| 0.7 – 1.0 | Strong alignment ✅ |
| 0.3 – 0.6 | Moderate — review rubric |
| < 0.3 | Weak judge |

---

## 🏷️ Scoring Rubric (Open-Ended)

| Score | Label | Description |
|---|---|---|
| 5 | **Excellent** | Matches or exceeds LLM reference |
| 4 | **Good** | Clinically sound, minor omissions |
| 3 | **Partial** | Acceptable but lacks precision |
| 2 | **Weak** | Major clinical omission |
| 1 | **Critical** | Hallucination or dangerous error |

MCQ questions are graded **deterministically** (model letter vs. gold answer — no API call).

---

## 🚩 Error Flag Taxonomy

| Flag | Meaning |
|---|---|
| `PROMPT_LEAKAGE` | Model repeated instructions from the prompt |
| `HALLUCINATION` | Fabricated clinical information |
| `OVER_VERBOSE` | Excessively long response that misleads the judge |
| `FACTUAL_ERROR` | Medically incorrect statement |
| `PARTIAL_ANSWER` | Incomplete response |
| `OFF_TOPIC` | Irrelevant content |

---

## 📦 Research Export

Two export formats are available in `exports/`:

- **JSONL** (`medeval-research-2026-05-14.jsonl`) — for ML pipelines and scripting
- **CSV** (`medeval-research-2026-05-14.csv`) — for Excel / SPSS / statistical tools

These exports contain only research-relevant data: questions, SLM responses, judge scores, human scores, and flags. Operational data (users, logs, API keys) is excluded.

---

## 🔒 Security Notes

- API keys are stored privately per user and deleted on logout — never included in backups.
- All evaluations are authenticated via JWT.
- Backup files contain evaluation data only — store securely.

---

## 📚 References

- Zheng et al. *Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena*. arXiv:2306.05685, 2023.
- Jiang et al. *Mistral 7B*. arXiv:2310.06825, 2023.
- Manes et al. *K-QA: A Real-World Medical Q&A Benchmark*. arXiv:2401.14493, 2024.
- Kung et al. *Performance of ChatGPT on USMLE*. PLOS Digital Health, 2(2), 2023.
- Spearman, C. *The Proof and Measurement of Association between Two Things*. AJP, 1904.
