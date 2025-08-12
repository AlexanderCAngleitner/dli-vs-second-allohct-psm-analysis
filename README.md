# DLI vs. second allo-HCT after post-transplant relapse
Survival analyses (KM, Cox), propensity-score matching (PSM), and competing-risks (Fine–Gray)

## Overview
This repository contains code and documentation to reproduce the comparative analyses of therapeutic donor lymphocyte infusion (DLI) vs. **second allogeneic hematopoietic cell transplantation** (second allo-HCT) in patients relapsing after a first allo-HCT. It covers:

- Kaplan–Meier survival estimation and log-rank tests  
- Univariable/multivariable Cox proportional hazards models  
- Propensity-score matching (PSM)  
- Competing-risks analysis (Fine–Gray for RRM/NRM)

> **Note on censoring:** Analyses censor at the date of **crossover** to an alternative salvage therapy in either direction (DLI→second allo-HCT; second allo-HCT→DLI; second→third allo-HCT).

---

## Repository contents
- `Git_Hub_2nd.ipynb` – end-to-end analysis notebook (Python; calls R for Fine–Gray)
- `Requirements_2nd.txt` – Python packages
- `Requirements_R_2nd.txt` – R packages
- `LICENSE` – MIT license

---

## Requirements

### Python (tested on 3.11)
```bash
pip install -r Requirements_2nd.txt
```

### R (tested on 4.5.1)
Install the packages listed in `Requirements_R_2nd.txt`, e.g.:
```r
install.packages(c("cmprsk","survival","readxl","dplyr"))
```

---

## Data availability & privacy (GDPR/DSGVO)

**No patient-level data are distributed in this repository.** Due to data protection regulations and ethics approvals, raw clinical datasets cannot be shared. The repository provides **analysis code only**.  
Users who wish to reproduce the analyses must use their **own de-identified dataset** (see schema below) and ensure **compliance with applicable laws and institutional policies**, including the **EU General Data Protection Regulation (GDPR / DSGVO)**. Where required, obtain ethics committee/IRB approval and appropriate data-use agreements before processing personal data.

> If you need to test the workflow without patient data, consider creating a **synthetic example dataset** that follows the schema but contains no real patient information.

---

## Data schema
Place an Excel file named **`your_data.xlsx`** in the project root. Expected columns / coding:

| Column            | Type    | Description |
|-------------------|---------|-------------|
| `Treatment`       | int     | 0 = therapeutic DLI, 1 = second allo-HCT |
| `Time`            | numeric | Follow-up time **in years** to event/censoring |
| `Survival`        | int     | 1 = death, 0 = censored |
| `ECOG`            | int     | ECOG performance status (0–4) at intervention |
| `Time to Relapse` | numeric | Time from **first** allo-HCT to relapse (use one consistent unit, e.g., months) |
| `event`           | int     | For competing risks: 0 = censored, 1 = relapse-related mortality (RRM), 2 = non-relapse mortality (NRM) |
| `pair_id` (opt.)  | int     | Pair identifier (created after matching; used for stratified Cox / clustered SE) |

> If your column names differ, adjust them in the notebook before running.

---

## How to run
1. Ensure Python and R dependencies are installed (see **Requirements**).
2. Place **`your_data.xlsx`** in the repository root.
3. Open **`Git_Hub_2nd.ipynb`** and run all cells top-to-bottom.

**Outputs**
- Matched datasets (e.g., `matched_model_C.xlsx`, `matched_model_C_full.xlsx`)
- Kaplan–Meier plots (TIFF/PNG)
- Model summaries (Cox, Fine–Gray)

Default output locations are the working directory; adjust paths in the notebook if needed.

---

## Methods summary (as implemented)

### Propensity-score matching (PSM)
- **Propensity model:** logistic regression using **ECOG** and **time from first allo-HCT to relapse** (standardized).
- **Matching:** **1:1 nearest neighbor on the logit(PS)**, **without replacement**, **greedy** implementation, **caliper = 0.2 × SD(logit(PS))**.
- **Complete-case matching:** only rows with non-missing matching covariates are eligible.
- **Reproducibility:** a fixed random seed is set before matching to stabilize greedy ordering.
- **Balance diagnostics:** standardized mean differences (SMD), with |SMD| < 0.10 considered acceptable (Love plot shown in the notebook).
- **Post-match analyses:** models account for pairing (e.g., **Cox stratified by `pair_id`**; Fine–Gray with **cluster-robust SE** by `pair_id` where applicable).

### Survival & regression
- **OS:** Kaplan–Meier estimation; log-rank test for group comparison.
- **Cox PH models:** univariable and multivariable; proportional-hazards assumption evaluated via Schoenfeld residuals.
- **Competing risks (Fine–Gray):** subdistribution hazard models for RRM (**failcode = 1**) and NRM (**failcode = 2**), with **cencode = 0**.
- **RMST:** reported up to a prespecified truncation time (e.g., τ = 2 years) as a complement to hazard-based metrics.

---

## Reproducibility tips
- Use the provided requirements files (or pin exact versions).
- Keep a fixed random seed for matching (documented in the notebook).
- If you modify column names or units, update the mapping cells accordingly.

---

## License
This project is released under the **MIT License**. See **[LICENSE](LICENSE)** for details.

## Citation
If you use this repository or adapt parts of it, please cite the manuscript and/or this repository. (Optional: add a `CITATION.cff` for GitHub’s citation panel.)
