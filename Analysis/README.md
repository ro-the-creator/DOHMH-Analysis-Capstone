# Analysis — NYC DOHMH Emergency Department Data

This folder contains two Jupyter notebooks that together form the full analytical pipeline for the NYC Department of Health and Mental Hygiene (DOHMH) Emergency Department dataset, covering the COVID-19 pandemic period.

---

## Notebooks

### 1. `cleaning-pipeline.ipynb` — ETL Pipeline

**Last updated:** March 31, 2026

Reads the raw DOHMH CSV, applies a structured cleaning and feature engineering pipeline, and outputs a cleaned CSV ready for analysis.

#### Source Dataset

| Field | Detail |
|---|---|
| **Coverage** | March 1, 2020 – December 31, 2021 (COVID-19 Pandemic) |
| **Raw row count** | ~12.9 million rows |
| **Raw file** | `Data/Emergency_Department_Visits_and_Admissions_for_Influenza-like_Illness_and_or_Pneumonia_20250714.csv` |
| **Output file** | `Data/nyc_dohmh_clean.csv` |

#### Raw Schema

| Column | Description | Raw Type |
|---|---|---|
| `extract_date` | Date of data extraction | Floating Timestamp |
| `date` | Date of emergency department visit | Floating Timestamp |
| `mod_zcta` | Modified ZIP Code Tabulation Area of patient residence | Text |
| `total_ed_visits` | Count of all ED visits | Number |
| `ili_pne_visits` | Count of influenza-like illness and/or pneumonia ED visits | Number |
| `ili_pne_admissions` | Count of ILI/pneumonia visits admitted to hospital | Number |

#### Cleaning & Transformation Steps

| Step | Column | Transformation |
|---|---|---|
| 1 | `extract_date` | Dropped entirely |
| 2 | `date` | Drop missing → enforce `datetime` (`mm/dd/yyyy`) |
| 3 | `mod_zcta` | Drop missing → convert to integer → zero-pad to 5-digit string |
| 4 | `borough` *(engineered)* | Map `mod_zcta` to NYC borough; drop rows with no match |
| 5 | `visit_admissions_ratio` *(engineered)* | `ili_pne_visits / total_ed_visits`; rows where ratio = 0 dropped |
| 6 | `total_ed_visits`, `ili_pne_visits`, `ili_pne_admissions` | Drop missing → IQR outlier removal → cast to `int64` |
| 7 | `visit_admissions_ratio` | IQR outlier removal → keep as `float` |

**Borough mapping** uses ZIP code ranges:

| Borough | ZIP Code Range(s) |
|---|---|
| Manhattan | 10001–10281 |
| The Bronx | 10451–10474 |
| Brooklyn | 11201–11255 |
| Queens | 11004–11109, 11351–11697 |
| Staten Island | 10301–10313 |

#### Usage

Run the single pipeline cell and provide paths when prompted:

```
Enter the path to the raw CSV file:
  → ../../Data/Emergency_Department_Visits_and_Admissions_for_Influenza-like_Illness_and_or_Pneumonia_20250714.csv

Enter the path to save the cleaned CSV file:
  → ../../Data/nyc_dohmh_clean.csv
```

#### Dependencies

```python
import pandas as pd
import numpy as np
```

---

### 2. `statistical-testing.ipynb` — Statistical Testing & EDA

Performs exploratory data analysis and formal hypothesis testing on the cleaned dataset to answer the core research questions.

#### Research Questions

1. **Which borough has the most patient visits turning into hospital admissions?**
   — i.e., which borough is experiencing the most severe illness cases?

2. **Which neighborhoods are high in visits but low in admissions, potentially needing more support?**
   — This could indicate hospital overflow or access issues.

> NOTE: 
> Visits and admissions in this dataset are strictly for influenza/pneumonia symptoms, which were major COVID-19 indicators during the study period.

---

#### Analysis Workflow

##### Step 1 — Pre-Check Diagnostics

Before running statistical tests, three diagnostics are applied to validate assumptions:

| Diagnostic | Purpose | Finding |
|---|---|---|
| Pipeline validation | Confirm no zero values in `visit_admissions_ratio` | Passed — no zeros present |
| Distribution check | Histogram + KDE per borough | All 5 boroughs are right-skewed; CLT applies given large *n* |
| Levene's Test | Assess equality of variances across boroughs | Reject H₀ — variances are statistically significantly unequal |

##### Step 2 — Statistical Test Selection

Because of **unequal variances** and **unequal sample sizes** across boroughs, standard one-way ANOVA is not appropriate. The analysis uses:

- **Welch's ANOVA** — robust to unequal variances and sample sizes
- **Games-Howell Post-Hoc Test** — pairwise comparisons that do not assume equal variance or equal *n*

##### Step 3 — Welch's ANOVA

**Target variable:** `visit_admissions_ratio`  
**Grouping variable:** `borough`

| Hypothesis | Statement |
|---|---|
| H₀ | There is no statistically significant difference in the mean visit:admissions ratio between boroughs |
| H₁ | One or more boroughs have a statistically significant difference in their mean visit:admissions ratio |

##### Step 4 — Games-Howell Post-Hoc Test

If Welch's ANOVA rejects H₀, Games-Howell pairwise comparisons identify *which* borough pairs differ significantly. Mean ratios per borough are also reported and ranked.

---

#### Dependencies

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import scipy.stats as stats
import pingouin as pg
```

---

## File Map

```
Analysis/
├── README.md                   ← this file
├── cleaning-pipeline.ipynb     ← ETL: raw CSV → nyc_dohmh_clean.csv
└── statistical-testing.ipynb   ← EDA + Welch's ANOVA + Games-Howell
```

**Upstream data files** (referenced but stored in `Data/`):

| File | Role |
|---|---|
| `Data/Emergency_Department_Visits_and_Admissions_for_Influenza-like_Illness_and_or_Pneumonia_20250714.csv` | Raw input |
| `Data/nyc_dohmh_clean.csv` | Cleaned output used by statistical testing |

> !NOTE
> Data folder does not live in GitHub Repository since the file is too large. Instead, the main repository README.md contains the link the original [NYC OpenData](https://data.cityofnewyork.us/Health/Emergency-Department-Visits-and-Admissions-for-Inf/2nwg-uqyg/about_data) dataset source.