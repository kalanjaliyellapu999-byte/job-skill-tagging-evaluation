# AI-Based Job Description Skill Tagging Evaluation

**Internship Work Sample | AI/ML Quality & Evaluation Engineering**  
**Repository / Workspace:** `job_skill_tagging_evaluation/`  
**Execution Environment:** Windows (PowerShell / Command Prompt)  
**Author Identification:** Candidate Submission (Personal identifying information removed for double-blind review)

---

## 1. Project Introduction & Problem Framing

Automated skill extraction from unstructured job descriptions (JDs) is a critical core subsystem for modern university and early-career job placement boards. When candidates search for internships, matching engines rely on predicted skill tags to index opportunities and rank students.

In this operational setting, skill-tagging errors cause severe asymmetric failures:
1. **False Positives (Precision Failures):** Over-predicting non-essential, invalid, or hallucinated skills falsely inflates job prerequisites. Students possessing genuine core qualifications are unfairly filtered out or deterred from applying.
2. **False Negatives (Recall Failures):** Omitting core technical requirements creates search opacity. Qualified students searching by their core competencies (e.g., React, PostgreSQL, Power BI) never discover relevant openings, leaving vacancies unfilled.

This project delivers an **independent, reproducible, multi-label evaluation framework** to assess the accuracy, reliability, and failure modes of an existing AI skill-tagging model across 50 fictional job descriptions benchmarked against an official 54-skill taxonomy.

> **Project Scope:** This is strictly an AI model evaluation and quality auditing system. No model retraining or recommendation ranking algorithms were executed.

---

## 2. Input Datasets & Schema Analysis

The evaluation is conducted strictly using two original, unmodified CSV files located in `data/`:

### A. `data/job-descriptions-tagged.csv`
Contains 50 fictional job descriptions spanning entry-level internship roles (Data Analyst, Frontend Developer, Backend Developer, Content Writer, Marketing Intern, UI/UX Design Intern, Finance Intern, Machine Learning Intern, Customer Support Intern, HR Intern).

| Column Name | Data Type | Description |
| :--- | :--- | :--- |
| `jd_id` | String | Unique identifier (`JD001` to `JD050`). Exactly 50 unique rows. |
| `title` | String | Fictional internship role title. |
| `company` | String | Fictional hiring company name. |
| `description` | String | Unstructured text specifying responsibilities, required skills, optional skills, and culture. |
| `model_tags` | String | Semicolon-delimited skill tags predicted by the evaluated AI model. |

### B. `data/skill-list.csv`
The official, authoritative taxonomy defining the 54 permitted skills:

```
AWS, Accounting, Blockchain, C++, CSS, Canva, Content Writing, Copy Editing,
Copywriting, Customer Support, Data Cleaning, Design Systems, Docker,
Email Marketing, Figma, Financial Modelling, GST, Git, Google Analytics,
HR Operations, HTML, Interviewing, Java, JavaScript, Jira, Kubernetes,
Looker, Machine Learning, Microsoft Excel, Node.js, Pandas, PostgreSQL,
Power BI, Prototyping, PyTorch, Python, R, REST APIs, React, Recruiting,
Research, SAP, SEO, SQL, Social Media Marketing, Statistics, Tableau,
Tally, TypeScript, User Research, Wireframing, WordPress, Written English, Zendesk
```

*Note on Data Integrity:* Neither CSV file has been modified or tampered with.

---

## 3. Evaluation & Annotation Methodology

A multi-label classification paradigm is employed where each job description represents an instance evaluated against a reference set of required skills.

### A. Ground-Truth Annotation & Review-Status Mechanism
- **Automated Draft Extraction:** Reference required and optional skills are parsed deterministically from the unstructured job description text using regex-based linguistic boundary matching. While systematic and auditable, these represent **rule-generated draft annotations**.
- **Transparent Human Review Status:** In accordance with scientific integrity guidelines, automated draft labels are clearly distinguished from independent human-reviewed labels. Because an external, multi-annotator human panel review was not conducted, all 50 job descriptions are explicitly assigned:
  ```
  review_status: PENDING_HUMAN_REVIEW (Rule-generated draft; awaiting independent human audit)
  ```
  No claim of completed independent human verification is made.

### B. Multi-Label Classification Rules:
* **True Positive (TP):** A model-predicted skill that matches an explicitly required skill extracted from core duty clauses (*"You will use..."*, *"Required:"*, *"We need someone who knows..."*).
* **False Positive (FP):** A model-predicted tag that is not supported as a mandatory requirement. This includes:
  1. *Optional Skills Intrusion:* Skills from preferred clauses (*"Bonus if..."*, *"Nice to have:"*) promoted to mandatory tags.
  2. *Non-Taxonomy / Unmapped Tags:* Predictions outside the 54 official taxonomy skills (soft skills, colloquial expressions, or broad unmapped terms).
  3. *Hallucinated Skills:* Enterprise software tags predicted with zero textual support.
* **False Negative (FN):** An explicitly required technical skill present in the job description that the model failed to predict.
* **Modal Operator Separation:** Preferred or optional skills qualified by modal phrases (*"Bonus if you have used..."*, *"Nice to have:"*, *"is a plus, not a must"*) are strictly cataloged separately. They are excluded from required-skill False Negatives and do not contaminate the recall denominator.

---

## 4. Transparent Synonym Mapping Approach

The evaluated model frequently emits common abbreviations, shortened names, or regional spelling variants. To avoid artificially penalizing valid predictions while preserving taxonomy integrity, a strict, evidence-based synonym mapping is applied:

| Original Model Tag | Canonical Taxonomy Skill | Linguistic & Domain Justification |
| :--- | :--- | :--- |
| `Excel` | `Microsoft Excel` | Universal colloquial abbreviation for Microsoft Excel spreadsheet software. |
| `Financial Modeling` | `Financial Modelling` | American English spelling variant mapped to British English taxonomy standard. |
| `JS` | `JavaScript` | Standard software engineering acronym for JavaScript programming language. |
| `ML` | `Machine Learning` | Standard artificial intelligence acronym for Machine Learning. |
| `Node` | `Node.js` | Universal developer shorthand for the Node.js JavaScript runtime environment. |
| `Postgres` | `PostgreSQL` | Official and universally recognized abbreviation for PostgreSQL database. |

### Exclusion of Broad / Ambiguous Terms:
In this audited version, broad terms previously considered for mapping were evaluated and **deliberately excluded**:
* **`API` is NOT mapped to `REST APIs`:** "API" is a general software engineering concept that encompasses GraphQL, gRPC, SOAP, SDK interfaces, or internal library APIs. Automatically assuming "REST APIs" without explicit qualification makes ungrounded assumptions. As a result, when the model predicts "API", it is classified as a non-taxonomy False Positive, and any omitted "REST APIs" requirement is recorded as a False Negative.
* **`Social Media` is NOT mapped to `Social Media Marketing`:** "Social Media" can refer to content creation, community engagement, moderation, or personal presence. Mapping it directly to the specialized domain of "Social Media Marketing" lacks rigorous semantic equivalence.

### Soft Skills Handling:
Non-canonical soft skills predicted by the model (`Leadership`, `Teamwork`, `Problem Solving`, `Communication`, `Time Management`) are not in the official technical taxonomy and lack canonical technical mappings. They are properly classified as invalid predictions (False Positives).

---

## 5. Performance Metrics & Benchmark Results

All metrics are micro-averaged across the complete aggregated corpus of 50 job descriptions:

$$\text{Micro Precision} = \frac{\sum \text{TP}}{\sum \text{TP} + \sum \text{FP}} = \frac{150}{150 + 50} = \frac{150}{200} = \mathbf{75.00\%}$$

$$\text{Micro Recall} = \frac{\sum \text{TP}}{\sum \text{TP} + \sum \text{FN}} = \frac{150}{150 + 27} = \frac{150}{177} = \mathbf{84.75\%}$$

$$\text{Micro F1-Score} = \frac{2 \times \text{Precision} \times \text{Recall}}{\text{Precision} + \text{Recall}} = \frac{2 \times 0.7500 \times 0.8475}{0.7500 + 0.8475} = \mathbf{79.58\%}$$

$$\text{Required-Skill Miss Rate} = \frac{\sum \text{FN}}{\sum \text{TP} + \sum \text{FN}} = \frac{27}{150 + 27} = \frac{27}{177} = \mathbf{15.25\%}$$

### Summary Metric Table:

| Metric Name | Mathematical Formula | Raw Value | Percentage (%) | Operational Interpretation |
| :--- | :--- | :--- | :--- | :--- |
| **Total JDs Evaluated** | Count of Unique IDs | 50 JDs | 100.00% | Full benchmark corpus evaluated |
| **Reference Required Skills** | $\sum \text{Reference Required}$ | 177 skills | — | Ground-truth required skills across 50 JDs |
| **Model Predicted Tags** | $\sum \text{Predicted Tags}$ | 200 tags | — | Total unique tags emitted by model |
| **True Positives (TP)** | $\text{Predicted} \cap \text{Required}$ | **150** | — | Correctly extracted required skills |
| **False Positives (FP)** | $\text{Predicted} \setminus \text{Required}$ | **50** | — | Erroneous tags (20 optional + 18 non-tax + 12 hallucinated) |
| **False Negatives (FN)** | $\text{Required} \setminus \text{Predicted}$ | **27** | — | Missed mandatory technical requirements |
| **Micro Precision** | $\text{TP} / (\text{TP} + \text{FP})$ | $150 / 200$ | **75.00%** | Accuracy of emitted skill tags |
| **Micro Recall** | $\text{TP} / (\text{TP} + \text{FN})$ | $150 / 177$ | **84.75%** | Coverage of required skill requirements |
| **Micro F1-Score** | $2PR / (P + R)$ | Harmonic Mean | **79.58%** | Balanced multi-label quality score |
| **Required Miss Rate** | $\text{FN} / (\text{TP} + \text{FN})$ | $27 / 177$ | **15.25%** | Frequency of omitted essential skills |

---

## 6. Three Empirical Error Patterns

Analysis of the 77 total errors (50 FPs + 27 FNs) reveals exactly three major systemic failure modes:

```
Total Errors (77):
├── Pattern 1: Modality Insensitivity (20 errors, 25.97%)
├── Pattern 2: Domain Hallucinations & Non-Taxonomy Tags (30 errors, 38.96%)
│   ├── Non-Taxonomy / Unmapped Tags (18 errors)
│   └── Unrelated Enterprise Hallucinations (12 errors)
└── Pattern 3: Core Technical Skill Omissions (27 errors, 35.06%)
```

### Pattern 1: Modality Insensitivity (Over-tagging Optional Skills as Required)
* **Count / Share:** 20 occurrences (25.97% of total errors)
* **Underlying Mechanism:** The model detects skill entities but ignores contextual modal qualifiers (*"Bonus if you have used..."*, *"Nice to have:"*, *"is a plus, not a must"*), promoting 40.0% of optional skills into mandatory tags.
* **Verified JD Examples:**
  * **JD004 (Content Writer Intern):** Predicted `SEO` (Text states: *"Bonus if you have used SEO"*).
  * **JD011 (Data Analyst Intern):** Predicted `Python` (Text states: *"Bonus if you have used Python"*).
  * **JD022 (Frontend Developer Intern):** Predicted `Git` (Text states: *"Git is a plus, not a must"*).
  * **JD047 (Finance Intern):** Predicted `Accounting` (Text states: *"Nice to have: Accounting"*).
* **Job Matching Impact:** Creates an artificial barrier to entry. Early-career students who are fully capable of performing the core role are filtered out because non-essential "bonus" skills are treated as mandatory gates.

### Pattern 2: Domain-Incongruent Hallucinations & Non-Taxonomy Invasions
* **Count / Share:** 30 occurrences (38.96% of total errors)
* **Underlying Mechanism:** The model emits tags unsupported by the description text or permitted taxonomy:
  1. *Enterprise Hallucinations (12 instances):* Hallucinating complex enterprise tools (`SAP`, `Kubernetes`, `Blockchain`, `Machine Learning`) in non-technical postings (Finance, HR, Content Writing).
  2. *Non-Taxonomy & Unmapped Tags (18 instances):* Emitting conversational soft skills (`Leadership` 7x, `Problem Solving` 3x, `Teamwork` 3x, `Communication` 2x, `Time Management` 1x) or broad unmapped terms (`API` 1x, `Social Media` 1x) outside official taxonomy bounds.
* **Verified JD Examples:**
  * **JD004 & JD044 (Content Writer Intern):** Predicted `SAP` (zero text support).
  * **JD007 (Finance Intern) & JD025 (Marketing Intern):** Predicted `Kubernetes` (zero text support).
  * **JD018 (ML Intern) & JD030 (HR Intern):** Predicted `Blockchain` (zero text support).
  * **JD003 (Backend Developer Intern) & JD040 (HR Intern):** Predicted `Leadership` (outside taxonomy).
* **Job Matching Impact:** Causes severe recommendation distortion. Students searching for DevOps or enterprise ERP roles get served entry-level content or finance jobs, degrading user trust in the platform.

### Pattern 3: Omission of Core Required Technical Skills (False Negatives)
* **Count / Share:** 27 occurrences (35.06% of total errors; 15.25% miss rate)
* **Underlying Mechanism:** The model fails to extract essential technical competencies explicitly stated in daily duty sentences (*"You will use React, TypeScript, CSS and JavaScript every day"*), frequently dropping subsequent items in long compound lists.
* **Verified JD Examples:**
  * **JD002 (Frontend Developer Intern):** Missed `React` from daily tools.
  * **JD013 & JD043 (Backend Developer Intern):** Missed `PostgreSQL` and `REST APIs`.
  * **JD023 (Backend Developer Intern):** Missed `REST APIs` (predicted unmapped `API`).
  * **JD041 (Data Analyst Intern):** Missed `Power BI` from required tools.
  * **JD045 (Marketing Intern):** Missed `Copywriting` and `Social Media Marketing` (predicted unmapped `Social Media`).
* **Job Matching Impact:** Critical discoverability failure. Students possessing high-demand skills (React, PostgreSQL, Power BI) never see these listings when filtering by their primary technical strengths.

---

## 7. Priority Improvement Recommendation (Proposed Architecture)

### Proposed Architectural Design:
To resolve these failure modes without requiring complete model re-training, an engineering recommendation is formulated for a **Two-Stage Taxonomy Enforcement & Linguistic Modality Gating Pipeline** in post-processing:

```
[Raw Model Output Tags]
        │
        ▼
┌──────────────────────────────────────────────┐
│ Stage 1: Taxonomy Filter & Synonym Resolver  │ ── Discard non-taxonomy tokens & map synonyms
└──────────────────────────────────────────────┘
        │
        ▼
┌──────────────────────────────────────────────┐
│ Stage 2: Linguistic Modality Gate            │ ── Route "bonus/nice-to-have" skills to optional_tags,
└──────────────────────────────────────────────┘    preventing contamination of required_tags
        │
        ▼
┌──────────────────────────────────────────────┐
│ Stage 3: Lexical Evidence Grounding Verifier │ ── Suppress hallucinated enterprise tags lacking text support
└──────────────────────────────────────────────┘
        │
        ▼
[Filtered & Verified Required Skill Tags]
```

### Baseline Performance vs. Proposed Target Architecture (Theoretical Projection):

| Metric | Measured Baseline Model | Proposed Target (Theoretical) | Expected Impact Direction |
| :--- | :---: | :---: | :--- |
| **True Positives (TP)** | **150** | 150 | Preserve all legitimate required skills |
| **False Positives (FP)** | **50** | Reduced (Target: ~0) | Discard non-taxonomy tokens and route optional modals |
| **False Negatives (FN)** | **27** | 27 (Phase 1) | Requires secondary dictionary extraction to resolve |
| **Micro Precision** | **75.00%** | Substantially Higher | Elimination of modality intrusions and hallucinations |
| **Micro Recall** | **84.75%** | 84.75% | Preserves current recall baseline |
| **Micro F1-Score** | **79.58%** | Higher | Boosted by reduction in False Positives |
| **Required Miss Rate** | **15.25%** | 15.25% (Phase 1) | Addressed via Phase 2 dictionary-based fallback parser |

> **Evaluation Integrity Disclosure:** The post-gating pipeline is presented strictly as a **proposed architectural design recommendation**. In accordance with scientific integrity standards, no actual before-and-after improvements or 100% precision metrics are claimed as measured results, as the post-processing pipeline has not yet been implemented or empirically tested.

---

## 8. Instructions to Run on Windows

The evaluation pipeline is fully self-contained in `evaluation.py`.

### Prerequisites:
- Python 3.10+ installed on Windows.
- Standard libraries: `os`, `sys`, `re`, `zipfile`, `xml`.
- Third-party packages: `pandas`, `numpy`, `pymupdf`.

### Execution Commands:

Open PowerShell or Command Prompt, navigate to the project directory, and run:

```powershell
cd C:\Users\bavii\.gemini\antigravity\scratch\job_skill_tagging_evaluation

# Run the complete automated evaluation pipeline
python evaluation.py
```

### Automated Pre-Flight Validation Suite:
When executed, `evaluation.py` automatically executes 10 validation checks:
1. `[PASS 1/10]` Exactly 50 unique JD IDs verified (`JD001` to `JD050`).
2. `[PASS 2/10]` Zero duplicate JD IDs confirmed.
3. `[PASS 3/10]` All 200 original model predicted tags fully accounted for.
4. `[PASS 4/10]` Strict mutual exclusivity between TP, FP, and FN sets verified.
5. `[PASS 5/10]` Every required skill accounted for: (TP union FN == Required).
6. `[PASS 6/10]` Optional skills strictly excluded from required False Negatives (FN).
7. `[PASS 7/10]` Independent metric recalculation matches exact figures: Precision 75.00%, Recall 84.75%, F1 79.58%, Miss Rate 15.25%.
8. `[PASS 8/10]` Data-driven validation verified: TP=150, FP=50, FN=27 (Total Pred=200, Total Req=177).
9. `[PASS 9/10]` All 3 error patterns substantiated by real JD IDs (JD004, JD007, JD002, etc.).
10. `[PASS 10/10]` PDF document strictly confirmed to be exactly 1 page (1 page).

---

## 9. Description of Deliverable Files

```
job_skill_tagging_evaluation/
│
├── data/
│   ├── job-descriptions-tagged.csv     # Original 50 fictional JDs with model tags (unmodified)
│   └── skill-list.csv                  # Official 54-skill permitted taxonomy (unmodified)
│
├── charts/
│   ├── chart1_tp_fp_fn.png             # Visual count comparison of TP (150), FP (50), FN (27)
│   ├── chart2_metrics_summary.png      # Precision (75.00%), Recall (84.75%), F1 (79.58%), Miss Rate (15.25%)
│   └── chart3_error_distribution.png   # Error breakdown across the 3 empirical failure patterns (77 errors)
│
├── skill_evaluation.xlsx               # 4-sheet formatted Excel workbook
│   ├── Sheet 1: Evaluation Labels      # Detailed 50-JD audit with evidence, notes & review status
│   ├── Sheet 2: Synonym Mapping        # 6 canonical normalization lookup rules
│   ├── Sheet 3: Performance Summary    # Aggregated KPI metrics, counts & formulas
│   └── Sheet 4: Error Analysis         # Empirical pattern classification & JD examples
│
├── evaluation_report.pdf               # Strictly ONE-PAGE executive summary report
├── evaluation.py                       # Self-contained reproducible Python evaluation script
├── LICENSE                             # MIT Open Source License
└── README.md                           # Comprehensive methodology & technical documentation
```

---

## 10. Known Limitations & Tool Usage Disclosure

### Known Limitations:
1. **Synthetic Nature of Corpus:** The 50 fictional job descriptions exhibit uniform syntactic structures (*"We need someone who knows..."*, *"Bonus if you have used..."*). Real-world job postings from diverse employers feature varied colloquial phrasing, nested bullet points, and complex qualifications.
2. **Review Status Designation:** Reference annotations were produced via deterministic regex parsing and are designated as `PENDING_HUMAN_REVIEW`. True multi-annotator inter-rater reliability (such as Fleiss' or Cohen's Kappa) requires an independent external panel of human domain experts.

### AI Tool Usage Disclosure:
In accordance with internship assessment guidelines:
- **Assisted Scaffolding:** Antigravity AI was utilized as a productivity assistant to draft boilerplate Python code, OpenXML formatting, PyMuPDF vector drawing routines, and Markdown documentation layout.
- **Methodology & Correctness Audit:** All evaluation formulas, classification boundaries, empirical failure patterns, synonym constraints, and data-driven assertions were reviewed, tested, and validated by the author.
- **Zero Data Fabrication:** No dataset records, evaluation counts, empirical examples, or performance metrics were fabricated. All figures reflect verified calculations from the input datasets.
