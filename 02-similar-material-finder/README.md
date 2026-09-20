# SAP–PLM Material Entity Resolution & Duplicate Detection

Work in progress research prototype. Structured-attribute entity resolution has reached a validated baseline with a confidence-based human-review workflow. The no-attribute branch remains experimental, while exact-profile duplicate detection already produces actionable master-data cleanup candidates.

## Project overview

Master-data systems often contain incomplete descriptions, inconsistent units, differently formatted technical attributes, and multiple records that refer to the same or very similar material. This project addresses two related problems:

1. **SAP → PLM entity resolution:** generate plausible PLM candidates for each SAP fabric material and rerank them using text and structured technical evidence.
2. **PLM duplicate audit:** identify PLM records with repeated exact technical profiles and prioritize the groups that deserve manual cleanup review.

The main modeling scope is knitted, woven, and denim fabric materials. Footwear-upper and trim materials are excluded from the primary fabric pipeline using conservative description rules.
## Current Results

> **Status:** Work in progress. These results represent the current research baseline and are expected to change as the pipeline is refined.

### 1. Structured-Attribute Entity Resolution

For SAP materials with usable structured attributes, the candidate-generation stage retrieves the known PLM match for approximately **97.5%** of evaluated materials.

The current reranking model was evaluated on a held-out set of **687 materials**.

| Metric                     | Result |
| -------------------------- | -----: |
| Candidate Recall           | 97.53% |
| Top-1 Accuracy             | 48.18% |
| Top-3 Accuracy             | 64.19% |
| Top-5 Accuracy             | 71.91% |
| Top-10 Accuracy            | 80.49% |
| Mean Reciprocal Rank (MRR) |  0.591 |

Performance varies substantially across material groups, suggesting that some categories are intrinsically more ambiguous than others.

| Material Group |  Top-1 |  Top-3 |
| -------------- | -----: | -----: |
| KNIT           | 34.27% | 52.81% |
| WOVEN          | 63.08% | 75.27% |
| DENIM          | 63.46% | 82.69% |

### 2. Confidence-Based Decision Layer

Rather than forcing the model to make a prediction for every material, the current system separates predictions into confidence-based workflow categories.

On the untouched holdout set:

| Decision      | Coverage |                 Performance |
| ------------- | -------: | --------------------------: |
| AUTO          |   22.71% |      96.15% Top-1 precision |
| REVIEW        |   16.30% | 66.96% Top-1 / 77.68% Top-3 |
| AUTO + REVIEW |   39.01% |     88.43% workflow success |

The purpose of this layer is to support a human-in-the-loop workflow: high-confidence predictions can be handled automatically, while ambiguous cases are presented as ranked candidates for manual review.

### 3. Materials Without Structured Attributes

A separate fallback pipeline is being developed for SAP materials that do not contain sufficient structured attributes.

The current development-stage results are:

| Metric           | Result |
| ---------------- | -----: |
| Candidate Recall | 83.24% |
| End-to-End Top-1 | 12.06% |
| Top-3            | 17.06% |
| Top-5            | 19.71% |
| Top-10           | 28.53% |

These results indicate that candidate retrieval remains viable, but ranking is substantially harder when the system must rely mainly on material descriptions and inferred attributes.

Auxiliary description-based models currently achieve strong Top-3 performance for predicting missing technical attributes:

| Prediction Task  |   Top-1 |   Top-3 |
| ---------------- | ------: | ------: |
| Fabric Structure | ~87–91% | ~96–99% |
| Yarn Count       |  54.69% |  89.93% |

The next research step is to use these inferred attributes more effectively without causing excessive candidate-pool growth.

### 4. Duplicate PLM Profile Detection

The duplicate-detection analysis currently covers **32,361 PLM codes** and identifies **29,778 unique observed technical profiles**.

A total of **1,289 technical profiles** occur more than once, involving **3,872 PLM codes**. This corresponds to approximately **11.97%** of the analyzed PLM master records.

These records are treated as **duplicate candidates rather than confirmed duplicates**, because identical observed technical attributes do not necessarily imply that two business records are interchangeable.

The analysis has also identified a smaller set of high-priority cases where multiple PLM codes with the same strong technical profile are actively referenced by SAP materials. These groups are candidates for domain-expert review.

### Interpretation

The current results suggest that structured-attribute entity resolution is sufficiently mature to support a confidence-based human-review prototype. The main unresolved challenge is entity resolution for materials with missing attributes, particularly in highly ambiguous material groups.

An additional open question is whether some apparent model errors are caused by multiple PLM codes representing technically equivalent records. A planned **duplicate-aware evaluation** will therefore compare exact-code accuracy with technical-profile equivalence.


## Repository structure

```text
material-entity-resolution-project/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   └── README.md
└── notebooks/
    └── material_entity_resolution_duplicate_detection.ipynb
```

## Methodology

The notebook follows a staged record-linkage architecture:

- **Scope filtering:** separates the target fabric domain from footwear-upper and trim records.
- **Identifier normalization:** standardizes SAP material IDs and PLM codes without discarding alphanumeric identifiers.
- **Feature normalization:** handles weight scale differences, units, yarn-count formats, fabric structure, coloring, weave type, and yarn type.
- **Candidate generation:** combines character n-gram TF-IDF retrieval with structured `structure + weight` and `fiber + weight` channels.
- **Pairwise reranking:** constructs one row per SAP–PLM pair, samples hard negatives, and compares interpretable Logistic Regression with Random Forest rerankers.
- **Leakage-aware evaluation:** splits at the SAP material level into train, validation, and untouched holdout partitions.
- **Operational decision layer:** routes recommendations into `AUTO`, `REVIEW`, or `LOW_CONFIDENCE` based on validation-selected score margins.
- **No-attribute fallback:** uses multiple text channels and description-derived pseudo-attributes for SAP materials that lack structured characteristics.
- **PLM duplicate audit:** hashes normalized technical profiles, then evaluates repeated profiles using technical completeness, status, and SAP usage.

## Data

The source data is **not committed** to this repository because it may contain proprietary enterprise master data. Place the following files in `data/` before running the notebook:

```text
data/
├── MaterialAttributes_full.xlsx
├── PLM Çalışması - Mara.xlsx
├── PLM_Codes.xlsx
└── PLM_MULTI_VAL_CHAR.xlsx
```

Alternatively, set the `DATA_FOLDER` environment variable to a directory containing those files.

The notebook intentionally retains some Turkish source-system column names because they are exact schema fields in the exported Excel files. Derived variables, comments, explanations, and project documentation are written in English.

## Installation

Create and activate a virtual environment, then install the project dependencies:

```bash
python -m venv .venv
```

Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

macOS / Linux:

```bash
source .venv/bin/activate
pip install -r requirements.txt
```

Launch Jupyter and open the notebook:

```bash
jupyter lab
```

## Evaluation philosophy

Candidate generation and reranking are evaluated separately. This distinction matters because a reranker cannot recover the true PLM record if the candidate-generation stage never retrieves it. The notebook therefore reports both candidate recall and rank-based metrics such as Top-K accuracy and mean reciprocal rank (MRR).

Confidence thresholds are selected on validation data and then evaluated on an untouched holdout split. The operational layer is designed to automate only high-confidence matches while routing ambiguous cases to human review.

## Important limitations

- Results depend on private enterprise data and cannot be reproduced from this public repository alone.
- Scope keywords and material-group mappings are domain-specific business assumptions.
- Weight scaling rules are inferred from observed SAP/PLM patterns and should be monitored for new source-system conventions.
- The no-attribute fallback is substantially more difficult because descriptions can be sparse or ambiguous.
- Repeated exact PLM technical profiles are **review candidates**, not automatic proof that two records should be merged.

## Suggested next improvements

A stronger public version of the project would add a synthetic sample dataset, move reusable functions into a `src/` package, add automated unit tests, and expose a small inference function that returns the top PLM candidates and confidence decision for a new SAP material.

## AI assistance disclosure

ChatGPT was used as a coding and documentation assistant during this project, including debugging, refactoring. The domain assumptions, data interpretation, modeling decisions, and final review remain the author's responsibility.
