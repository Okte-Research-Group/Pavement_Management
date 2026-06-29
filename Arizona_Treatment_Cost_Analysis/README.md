# Arizona Pavement Treatment Unit Cost Analysis

This folder contains the analysis code for estimating **pavement treatment unit cost distributions** from ADOT bid tab data, developed as part of the FHWA EAR project:

> **"Cost-Efficient Network-Level Pavement Management Framework for Flexible Pavement Preservation and Maintenance"**


The resulting cost distributions (lognormal parameters and correlation matrix) are used as inputs to the stochastic network-level model in [Arizona\_Case\_Study\_Stochastic](../Arizona_Case_Study_Stochastic/).

---

## Overview

The notebook processes ADOT bid tab records to produce **empirical and lognormal-fitted cost distributions** for each pavement treatment type used in the Arizona statewide network. All unit prices are inflation-adjusted to **2025 Q1 real dollars** using the National Highway Construction Cost Index (NHCCI) before any analysis.

For each treatment, the notebook computes:
- Unit price distributions by county and statewide (violin plots, boxplots)
- Cumulative distribution functions — empirical and lognormal fit
- Bootstrap-based summary statistics (sample size, mean, coefficient of variation)
- **Composite lane-mile costs** where individual pay items are combined using ADOT quantity assumptions

The lognormal summary table (mean, CoV, μ\_ln, σ\_ln) produced at the end of the notebook feeds directly into the Monte Carlo sampling step of the stochastic case study.

---

## Treatments Analyzed

### Pay-item unit price analyses
These sections analyze the bid unit price (e.g., $/ton, $/SY) directly from the bid records.

| Treatment | Pay Items Analyzed | Unit |
|-----------|-------------------|------|
| Asphalt Concrete (AC) | Binder, Mixture, Mineral Admixture | $/ton |
| Friction Course / Rubberized AC (FR) | Binder, Mixture, Mineral Admixture | $/ton |
| AC & FR Binder Content | Binder content derived from binder and mixture quantities | % by weight |
| Milling | Cold plane milling | $/SY |

### Composite lane-mile cost analyses
These sections combine multiple pay items into a single lane-mile cost using ADOT quantity assumptions.

| Treatment | Components | Unit |
|-----------|-----------|------|
| Remove & Replace 0.5-inch AC + FR | AC Mixture + FR Mixture + Milling + Tack Coat | $/lane-mile |
| Chip Seal | Bituminous cover + aggregate | $/lane-mile |
| Microsurfacing | Emulsion + aggregate | $/lane-mile |
| Crack Seal | Sealant material | $/lane-mile |
| Fog Coat | Rejuvenating emulsion | $/lane-mile |
| Mill + Cape Seal | Milling + Chip Seal + Microsurfacing | $/lane-mile |
| Remove & Repair 3-inch AC + Microsurfacing | Spot repair AC + Microsurfacing overlay | $/lane-mile |
| Reconstruction | AC + FR + Milling + Tack Coat + sub-base components | $/lane-mile |

---

## Notebook Structure

The notebook (`Arizona_Treatment_Cost_Analysis.ipynb`) runs end to end in section order:

1. **Setup** — reproducibility seed, library imports
2. **Inflation adjustment** — apply NHCCI deflator; output column `UnitPrice_Real_2025Q1`
3. **Data filtering** — retain Position == 1 records; deduplicate by job
4. **AC pay items** — unit price distributions for Binder, Mixture, Mineral Admixture, and Binder Content
5. **FR pay items** — same sub-sections as AC
6. **Milling** — unit price distributions and lane-mile cost estimates
7. **Remove & Replace 0.5-inch AC+FR** — composite lane-mile cost from AC, FR, and Milling; bootstrap distributions; violin and CDF by county
8. **ALL treatments combined** — side-by-side lane-mile cost comparison across all treatment types
9. **Individual surface treatments** — Chip Seal, Microsurfacing, Crack Seal, Fog Coat, Mill + Cape Seal; each with STATE-level violin, CDF, and bootstrap statistics
10. **Remove & Repair 3-inch AC + Microsurfacing** — spot-repair composite cost; STATE-level distribution and CDF
11. **Reconstruction** — component-level and composite STATE-level reconstruction cost distribution
12. **Presentation summary** — all-treatment overlay plots and lognormal summary table (mean, CoV, μ\_ln, σ\_ln) for every treatment
13. **Paired bootstrap correlation** — Spearman correlation between Remove & Replace AC and FR costs; outputs the correlation matrix used by the stochastic model

---

## Data

### Source

The bid tab data are obtained from **BidTabs.NET**, a commercial platform that aggregates highway construction bid records from state DOTs. The data cannot be shared publicly due to the platform's licensing terms. Researchers wishing to replicate the analysis should obtain a BidTabs.NET subscription or request the dataset from the corresponding author.

### Input Files

The notebook reads two Excel files:

**`ByJob.xlsx`** — pay-item level bid records, one row per (project × pay item × contractor).

| Column | Description |
|--------|-------------|
| `JobID` | ADOT project identifier |
| `ContractorID` | Winning contractor identifier |
| `ContractorName` | Winning contractor name |
| `PayItemID` | ADOT pay item code (e.g., `4030001` = AC Mixture) |
| `PayItemDescription` | Human-readable pay item name |
| `Quantity` | Bid quantity in the pay item unit |
| `Amount` | Unit bid price (nominal dollars) |
| `Extension` | Total pay item cost — `Quantity × Amount` |
| `Position` | Bid line position (1 = primary/winning bid) |
| `UnitEnglishID` | Unit of measure (e.g., TON, S.Y., GAL) |
| `CategoryID` / `CategoryDescription` | ADOT bid category |

**`PISearch_project_summary.xlsx`** — project-level metadata, one row per project.

| Column | Description |
|--------|-------------|
| `ProjectID` | ADOT project identifier (joins to `JobID` in ByJob) |
| `Job Desc` | Project description / title |
| `Job Size` | Total contract value ($) |
| `Bid Date` | Letting / bid opening date |
| `County` | Arizona county where the project is located |
| `Region` | ADOT administrative region (1 = Maricopa, 2 = South, 3 = North) |
| `JobFederalID` | Federal project number (e.g., STP, NH, ARRA) |
| `PopulationArea` | Area classification — Urban, Suburban, or Rural |

### NHCCI Deflation

The inflation adjustment uses the National Highway Construction Cost Index (NHCCI) table from FHWA's December 2025 quarterly release (`NHCCI_20251220.csv`). The base period is **2025 Q1**. The NHCCI file is included in this folder.

---

## Installation

Python 3.10 or later is recommended.

```bash
pip install pandas numpy matplotlib scipy
```

All imports are standard scientific Python libraries available on PyPI. No proprietary packages are required.

### Reproducibility

The notebook uses a global random seed (`GLOBAL_SEED = 42`) for all bootstrap draws. Fully identical results across kernel restarts also require a fixed `PYTHONHASHSEED`. A `.env` file in this folder sets `PYTHONHASHSEED=0`; VS Code loads it automatically. After adding or editing the `.env` file, **restart the kernel and Run All**.

---

## Running

1. Place `ByJob.xlsx` and `PISearch_project_summary.xlsx` in the same folder as the notebook.
2. Launch Jupyter and open `FHWA_Bid.ipynb`:
   ```bash
   jupyter lab
   ```
3. Run all cells top to bottom (**Kernel → Restart & Run All**).

Each section is self-contained except where noted (the Presentation summary and Correlation sections require lane-mile cost variables computed in earlier sections).

---

## Method Notes

- **Lane-mile cost construction.** Composite lane-mile costs are built from individual pay item unit prices using standard ADOT quantity assumptions (lane width, lift thickness, material densities). These assumptions are documented in the code comments of each relevant cell.
- **Bootstrap statistics.** All summary statistics (mean, CoV) are computed from a bootstrap sample of job-level median prices to account for multiple pay items within a single project. Bootstrap sample size is 10,000 draws with replacement.
- **Lognormal fitting.** The lognormal CDF is fit by method of moments (matching the empirical mean and standard deviation), not maximum likelihood. This matches the parameterization used in the stochastic case study.
- **County filtering.** Only counties with at least 5 job observations for a given pay item are shown in county-level plots. The STATE aggregate uses all available records.
- **Section 508 accessibility.** All plots use hatch patterns in addition to color to distinguish groups, and summary statistics are presented in accessible formatted tables.

---

## Data & Code Availability

- **Code:** available in this repository.
  - GitHub: [Pavement Management](https://github.com/Okte-Research-Group/Pavement_Management)
- **Bid tab data:** obtained from BidTabs.NET and cannot be shared publicly due to licensing. Contact the corresponding author for access inquiries.
- **NHCCI index:** publicly available from FHWA.

---

## Acknowledgments

This work is supported by the Federal Highway Administration (FHWA) under the project *Cost-Efficient Network-Level Pavement Management Framework for Flexible Pavement Preservation and Maintenance*. The authors acknowledge the FHWA team led by Sivaneswaran Nadarajah, and the support of the Arizona Department of Transportation (ADOT).

---
