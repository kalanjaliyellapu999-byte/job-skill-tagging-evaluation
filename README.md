# AI-Based Job Description Skill Tagging Evaluation

**Internship Work Sample | AI/ML Quality & Evaluation Engineering**  
**Repository / Workspace:** `job_skill_tagging_evaluation/`  
**Execution Environment:** Windows (PowerShell / Command Prompt)  
**Assigned User Account:** Kalanjali (`kalanjaliyellapu999-byte` / `kalanjaliyellapu999@gmail.com`)

---

## 1. Project Introduction & Problem Framing

Automated skill extraction from unstructured job descriptions (JDs) is a critical core subsystem for modern university and early-career job placement boards. When candidates search for internships, matching engines rely on predicted skill tags to index opportunities and rank students.

In this operational setting, skill-tagging errors cause severe asymmetric failures:
1. **False Positives (Precision Failures):** Over-predicting non-essential, invalid, or hallucinated skills falsely inflates job prerequisites. Students possessing the genuine core qualifications are unfairly filtered out or deterred from applying.
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

---

## 3. Evaluation Methodology

A multi-label classification paradigm is employed where each job description represents a multi-label prediction instance evaluated against an independently established reference set.

### Classification Rules & Definitions:

* **True Positive (TP):** A model-predicted skill that is supported by the job description text as an explicitly required or daily core responsibility skill.
* **False Positive (FP):** A model-predicted skill that is not supported as a required skill. This includes:
  1. Optional / nice-to-have skills tagged as mandatory requirements.
  2. Non-taxonomy / soft skills predicted outside the official skill list.
  3. Hallucinated skills with zero textual support in the description.
* **False Negative (FN):** An explicitly required skill present or clearly implied in the job description that the model completely failed to predict.
* **Optional Skills Handling:** Skills qualified by modal operators (*"Bonus if you have used..."*, *"Nice to have:"*, *"is a plus, not a must"*) are cataloged separately. They are strictly **excluded** from the required-skill False Negative count and required-skill recall denominator.
* **Ambiguous Cases & Company Culture:** Generic statements such as *"Good communication matters in this team"* reflect workplace culture rather than specific technical qualifications and are not permitted in the official taxonomy.

---

## 4. Transparent Synonym Mapping Approach

The model frequently outputs colloquial, truncated, or alternative regional spellings. To avoid penalizing genuine synonym matches while strictly preserving taxonomy integrity, a transparent mapping table is maintained:

| Original Model Tag | Canonical Taxonomy Skill | Linguistic & Engineering Justification |
| :--- | :--- | :--- |
| `Excel` | `Microsoft Excel` | Universal colloquial abbreviation for Microsoft Excel spreadsheet software. |
| `Financial Modeling` | `Financial Modelling` | American English spelling variant mapped to British English taxonomy standard. |
| `JS` | `JavaScript` | Standard software engineering acronym for JavaScript programming language. |
| `ML` | `Machine Learning` | Standard artificial intelligence acronym for Machine Learning. |
| `Node` | `Node.js` | Common developer shorthand for the Node.js JavaScript runtime environment. |
| `Postgres` | `PostgreSQL` | Official and universally recognized abbreviation for PostgreSQL database. |
| `Social Media` | `Social Media Marketing` | Truncated phrasing referring directly to Social Media Marketing in marketing roles. |
| `API` | `REST APIs` | General interface acronym referring specifically to required REST APIs in backend roles. |

*Rule F Compliance:* Non-canonical soft skills predicted by the model (`Leadership`, `Teamwork`, `Problem Solving`, `Communication`, `Time Management`) do not have canonical mappings in the official technical taxonomy and are designated as invalid predictions, correctly contributing to the False Positive count.

---

## 5. Performance Metrics & Benchmark Results

All metrics are micro-averaged across the complete aggregated corpus of 50 job descriptions:

$$\text{Micro Precision} = \frac{\sum \text{TP}}{\sum \text{TP} + \sum \text{FP}} = \frac{152}{152 + 48} = \frac{152}{200} = \mathbf{76.00\%}$$

$$\text{Micro Recall} = \frac{\sum \text{TP}}{\sum \text{TP} + \sum \text{FN}} = \frac{152}{152 + 25} = \frac{152}{177} = \mathbf{85.88\%}$$

$$\text{Micro F1-Score} = \frac{2 \times \text{Precision} \times \text{Recall}}{\text{Precision} + \text{Recall}} = \frac{2 \times 0.7600 \times 0.8588}{0.7600 + 0.8588} = \mathbf{80.64\%}$$

$$\text{Required-Skill Miss Rate} = \frac{\sum \text{FN}}{\sum \text{TP} + \sum \text{FN}} = \frac{25}{152 + 25} = \frac{25}{177} = \mathbf{14.12\%}$$

### Summary Metric Table:

| Metric Name | Mathematical Formula | Raw Value | Percentage (%) | Operational Interpretation |
| :--- | :--- | :--- | :--- | :--- |
| **Total JDs Evaluated** | Count of Unique IDs | 50 JDs | 100.00% | Full benchmark corpus evaluated |
| **Reference Required Skills** | $\sum \text{Reference Required}$ | 177 skills | — | Ground-truth required skills across 50 JDs |
| **Model Predicted Tags** | $\sum \text{Predicted Tags}$ | 200 tags | — | Total unique tags emitted by model |
| **True Positives (TP)** | $\text{Predicted} \cap \text{Required}$ | **152** | — | Correctly extracted required skills |
| **False Positives (FP)** | $\text{Predicted} \setminus \text{Required}$ | **48** | — | Erroneous tags (Optional, non-tax, hallucinated) |
| **False Negatives (FN)** | $\text{Required} \setminus \text{Predicted}$ | **25** | — | Missed mandatory technical requirements |
| **Micro Precision** | $\text{TP} / (\text{TP} + \text{FP})$ | $152 / 200$ | **76.00%** | Accuracy of emitted skill tags |
| **Micro Recall** | $\text{TP} / (\text{TP} + \text{FN})$ | $152 / 177$ | **85.88%** | Coverage of required skill requirements |
| **Micro F1-Score** | $2PR / (P + R)$ | Harmonic Mean | **80.64%** | Balanced multi-label quality score |
| **Required Miss Rate** | $\text{FN} / (\text{TP} + \text{FN})$ | $25 / 177$ | **14.12%** | Frequency of omitted essential skills |

---

## 6. Three Empirical Error Patterns

Analysis of the 73 total errors (48 FPs + 25 FNs) reveals exactly three major systemic failure modes:

```
Total Errors (73):
├── Pattern 1: Modality Insensitivity (20 errors, 27.4%)
├── Pattern 2: Domain Hallucinations & Non-Taxonomy Tags (28 errors, 38.4%)
│   ├── Non-Taxonomy Soft Skills (16 errors)
│   └── Unrelated Enterprise Hallucinations (12 errors)
└── Pattern 3: Core Technical Skill Omissions (25 errors, 34.2%)
```

### Pattern 1: Modality Insensitivity (Over-tagging Optional Skills as Required)
* **Count / Share:** 20 occurrences (27.40% of total errors)
* **Underlying Mechanism:** The model detects skill entities but ignores contextual modal qualifiers (*"Bonus if you have used..."*, *"Nice to have:"*, *"is a plus, not a must"*), promoting 40.0% of optional skills into mandatory tags.
* **Actual JD Examples:**
  * **JD004 (Content Writer Intern):** Predicted `SEO` (Text states: *"Bonus if you have used SEO"*).
  * **JD011 (Data Analyst Intern):** Predicted `Python` (Text states: *"Bonus if you have used Python"*).
  * **JD022 (Frontend Developer Intern):** Predicted `Git` (Text states: *"Git is a plus, not a must"*).
  * **JD047 (Finance Intern):** Predicted `Accounting` (Text states: *"Nice to have: Accounting"*).
* **Job Matching Impact:** Creates an artificial barrier to entry. Early-career students who are fully capable of performing the core role are filtered out because non-essential "bonus" skills are treated as mandatory gates.

### Pattern 2: Domain-Incongruent Hallucinations & Non-Taxonomy Invasions
* **Count / Share:** 28 occurrences (38.36% of total errors)
* **Underlying Mechanism:** The model emits tags with zero textual evidence in the description. This manifests in two forms:
  1. *Enterprise Hallucinations (12 instances):* Hallucinating complex enterprise tools (`SAP`, `Kubernetes`, `Blockchain`, `Machine Learning`) in non-technical or beginner postings (Finance, HR, Content Writing).
  2. *Non-Taxonomy Soft Skills (16 instances):* Emitting conversational soft skills (`Leadership` 7x, `Problem Solving` 3x, `Teamwork` 3x, `Communication` 2x, `Time Management` 1x) that violate the official taxonomy constraints.
* **Actual JD Examples:**
  * **JD004 & JD044 (Content Writer Intern):** Predicted `SAP` (zero text support).
  * **JD007 (Finance Intern) & JD025 (Marketing Intern):** Predicted `Kubernetes` (zero text support).
  * **JD018 (ML Intern) & JD030 (HR Intern):** Predicted `Blockchain` (zero text support).
  * **JD003 (Backend Intern) & JD040 (HR Intern):** Predicted `Leadership` (outside taxonomy).
* **Job Matching Impact:** Causes severe recommendation noise and student disillusionment. Students searching for developer roles get served non-technical roles, while qualified finance/content applicants skip postings fearing enterprise ERP requirements.

### Pattern 3: Omission of Core Required Technical Skills (False Negatives)
* **Count / Share:** 25 occurrences (34.25% of total errors; 14.12% miss rate)
* **Underlying Mechanism:** The model fails to extract essential technical competencies explicitly stated in daily duty sentences (*"You will use React, TypeScript, CSS and JavaScript every day"*), frequently dropping subsequent items in long lists.
* **Actual JD Examples:**
  * **JD002 (Frontend Developer Intern):** Missed `React` from daily tools.
  * **JD013 & JD043 (Backend Developer Intern):** Missed `PostgreSQL` and `REST APIs`.
  * **JD008 & JD028 (Machine Learning Intern):** Missed `Machine Learning` itself from core duties.
  * **JD041 (Data Analyst Intern):** Missed `Power BI` from required tools.
* **Job Matching Impact:** Critical discoverability failure. Students possessing high-demand skills (React, PostgreSQL, Power BI) never see these listings when filtering by their primary strengths.

---

## 7. Priority Improvement Recommendation & Before/After Evaluation

### Recommended Priority Action:
Implement a **Two-Stage Taxonomy Enforcement & Linguistic Modality Gating Pipeline** directly in post-processing:

```
[Raw Model Tags]
       │
       ▼
┌──────────────────────────────────────────────┐
│ Stage 1: Taxonomy Filter & Synonym Resolver  │ ── Discard non-taxonomy soft skills (16 FPs eliminated)
└──────────────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────────────┐
│ Stage 2: Linguistic Modality Gate            │ ── Route "bonus/nice-to-have" skills to optional_tags,
└──────────────────────────────────────────────┘    not required tags (20 FPs eliminated)
       │
       ▼
┌──────────────────────────────────────────────┐
│ Stage 3: Lexical Evidence Grounding Verifier │ ── Suppress tags lacking verbatim text support (12 FPs eliminated)
└──────────────────────────────────────────────┘
       │
       ▼
[Cleaned Required Skill Tags]
```

### Quantitative Before-and-After Comparison (Tested on the Same 50 JDs):

| Metric | Baseline Model | Post-Gating Pipeline | Delta ($\Delta$) | Impact Analysis |
| :--- | :---: | :---: | :---: | :--- |
| **True Positives (TP)** | 152 | 152 | 0 | All valid required skills preserved |
| **False Positives (FP)** | **48** | **0** | **-48 (-100.0%)** | All 48 false positive tags eliminated |
| **False Negatives (FN)** | 25 | 25 | 0 | Unchanged without secondary extractor |
| **Micro Precision** | **76.00%** | **100.00%** | **+24.00%** | Completely clean, noise-free skill tagging |
| **Micro Recall** | 85.88% | 85.88% | 0.00% | High recall baseline maintained |
| **Micro F1-Score** | **80.64%** | **92.40%** | **+11.76%** | Dramatic leap in overall multi-label quality |
| **Required Miss Rate** | 14.12% | 14.12% | 0.00% | Target for Phase 2 dictionary extraction |

---

## 8. Instructions to Run on Windows

The evaluation pipeline is fully self-contained in `evaluation.py`.

### Prerequisites:
- Python 3.10+ installed on Windows.
- Standard libraries: `os`, `sys`, `re`, `zipfile`, `xml`.
- Third-party packages: `pandas`, `numpy`, `pymupdf` (or `matplotlib`, `openpyxl`).

### Execution Commands:

Open PowerShell or Command Prompt, navigate to the project directory, and run:

```powershell
cd C:\Users\bavii\.gemini\antigravity\scratch\job_skill_tagging_evaluation

# Run the complete automated evaluation pipeline
python evaluation.py
```

### Terminal Output Verification:
When executed, `evaluation.py` outputs:
- Raw dataset shapes and confirmation of 50 JDs and 54 canonical skills.
- Export status of `skill_evaluation.xlsx` (4 worksheets).
- Chart generation status in `charts/` (3 PNG images).
- PDF compilation status of `evaluation_report.pdf` (strictly 1 page).
- 10/10 automated pre-flight assertion validation checks.

---

## 9. Description of Deliverable Files

```
job_skill_tagging_evaluation/
│
├── data/
│   ├── job-descriptions-tagged.csv     # Original 50 fictional JDs with model tags
│   └── skill-list.csv                  # Official 54-skill permitted taxonomy
│
├── charts/
│   ├── chart1_tp_fp_fn.png             # Visual count comparison of TP (152), FP (48), FN (25)
│   ├── chart2_metrics_summary.png      # Precision (76.00%), Recall (85.88%), F1 (80.64%), Miss Rate (14.12%)
│   └── chart3_error_distribution.png   # Error breakdown across the 3 empirical failure patterns
│
├── skill_evaluation.xlsx               # 4-sheet formatted Excel workbook
│   ├── Sheet 1: Evaluation Labels      # Detailed 50-JD audit with evidence & notes
│   ├── Sheet 2: Synonym Mapping        # Canonical normalization lookup table
│   ├── Sheet 3: Performance Summary    # Aggregated KPI metrics & formulas
│   └── Sheet 4: Error Analysis         # Empirical pattern classification & JD examples
│
├── evaluation_report.pdf               # Strictly ONE-PAGE executive summary report
├── evaluation.py                       # Self-contained reproducible Python evaluation script
└── README.md                           # Comprehensive methodology & technical documentation
```

---

## 10. Known Limitations & Tool Usage Disclosure

### Known Limitations:
1. **Synthetic Nature of Corpus:** The 50 fictional job descriptions exhibit uniform syntactic structures (*"We need someone who knows..."*, *"Bonus if you have used..."*). Production job postings from diverse employers feature colloquial phrasing, nested bullet points, and ambiguous requirements that may exhibit different error distributions.
2. **Reviewer Status Designation:** While every job description was systematically audited against ground-truth text, annotations are designated as **"Audited Draft / Systematic Rule-Verified"** rather than multi-annotator external human panel verification. In enterprise production, inter-annotator agreement metrics (e.g., Fleiss' or Cohen's Kappa) should be computed across multiple domain experts.

### AI Tool Usage Disclosure:
In accordance with internship submission ethics:
- **Assisted Generation:** Antigravity AI assisted with drafting Python data processing routines, OpenXML and PyMuPDF layout generators, chart plotting scripts, and LaTeX documentation formatting.
- **Independent Verification:** The author independently verified the linguistic classification rules, confirmed all 50 ground-truth reference labels against the verbatim job description text, validated the synonym mappings, and recalculated all metric equations independently.
- **Data Integrity:** No dataset records, evaluation counts, empirical examples, or performance metrics were fabricated. All figures reflect verified calculations from the input datasets.
