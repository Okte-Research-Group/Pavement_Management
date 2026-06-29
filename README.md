# Pavement_Management

Code repository for the FHWA EAR project:

> **"Cost-Efficient Network-Level Pavement Management Framework for Flexible Pavement Preservation and Maintenance"**

This repository contains the analysis code supporting the manuscript:

> **"Impact of Integrating Use Stage into Network-Level Life-Cycle Planning"**  
> S. U. Yildirim, S. Mostatab, M. Zeigham, E. Okte (corresponding author),  
> E. Tseng, I. L. Al-Qadi, and H. Ozer.  

---
## Overview

The repository implements a **network-level pavement life-cycle planning (LCP)**
framework that integrates both **agency costs** and **user costs** into
pavement management decisions. User costs are quantified as excess fuel
consumption (EFC) attributable to pavement roughness, estimated using the
Roughness Speed Impact (RSI) model (Ziyadi et al., 2018). The framework is
applied under both **deterministic** and **stochastic** (Monte Carlo) settings,
with treatment unit cost distributions derived from ADOT bid tab records and
inflation-adjusted to 2025 Q1 real dollars using the National Highway
Construction Cost Index (NHCCI).

Four folders are included, each in its own directory:

| Folder | Case Study | Data Source |
|--------|-----------|-------------|
| `Arizona Case Study - Deterministic LCCA/` | ADOT statewide highway network (deterministic) | ADOT HPMS 2022 shapefiles (public, on Zenodo) |
| `Arizona_Case_Study_Stochastic/` | ADOT statewide highway network (stochastic Monte Carlo) | ADOT HPMS 2022 shapefiles (public, on Zenodo) |
| `Arizona_Treatment_Cost_Analysis/` | ADOT bid tab treatment cost distributions | BidTabs.NET (restricted); NHCCI index (public) |
| `Minnesota Case Study/` | MnDOT state highway network | MnDOT CHIP dataset (restricted, dummy provided) |

---

## Repository Structure

```
Pavement_Management/
├── Arizona Case Study - Deterministic LCCA/
│   ├── Arizona_Case_Study_Deterministic_LCCA.ipynb
│   ├── requirements.txt
│   ├── README.md
│   └── adot_data/
│       └── gis/
│           ├── before_merge_0p1miles_arizona/
│           └── merged_5miles_arizona/
├── Arizona_Case_Study_Stochastic/
│   ├── Arizona_Case_Study_Stochastic.ipynb
│   ├── paired_rr_ac_fr_correlation_matrix.csv
│   ├── requirements.txt
│   ├── README.md
│   └── adot_data/
│       └── gis/
│           ├── before_merge_0p1miles_arizona/
│           └── merged_5miles_arizona/
├── Arizona_Treatment_Cost_Analysis/
│   ├── Arizona_Treatment_Cost_Analysis.ipynb
│   ├── NHCCI_20251220.csv
│   └── README.md
└── Minnesota Case Study/
    ├── minnesota_case_study.ipynb
    ├── mndot_dummy.xlsx
    └── README.md
```

---

## Case Studies

### Arizona Case Study (Deterministic)

A deterministic, network-level LCP model for the Arizona statewide highway
network. The model loads the ADOT pavement network, projects deterioration over
a 10-year horizon, applies the ADOT treatment decision tree, and allocates a
constrained annual budget under three prioritization strategies:

- **Worst-First** — most-distressed segments first
- **Preservation** — benefit/cost ratio
- **Traffic-Weighted Preservation** — benefit/cost weighted by AADT²

Agency costs and user costs (excess fuel from roughness) are reported for
each strategy. The run is fully deterministic and exactly reproducible.

Input data (ADOT shapefiles) are publicly available on Zenodo: `<10.5281/zenodo.20836630>`

See `Arizona Case Study - Deterministic LCCA/README.md` for full details.

---

### Arizona Case Study (Stochastic)

#### Network-Level Stochastic Simulation

Extends the deterministic Arizona framework to a fully stochastic setting using
Monte Carlo simulation. Uncertainty is propagated through two independent sources:
**pavement deterioration rates** (IRI, rutting, and cracking increments) and
**treatment unit costs**, each sampled once per realization to reflect systematic,
network-wide variability rather than independent segment noise. The model runs
*n* iterations and reports, for each of the three prioritization
strategies (Worst-First, Preservation, Traffic-Weighted Preservation), the full
probability distribution of outcomes — including mean trajectories and P10–P90
percentile bands for network condition metrics (IRI, rutting, cracking) and total
costs (agency and road-user).

Treatment unit cost distributions (lognormal parameters) are derived from the
companion Treatment Cost Analysis notebook described below. Remove & Replace costs
can optionally be sampled from a correlated joint distribution, whose Spearman
correlation matrix is also produced by that notebook.

Input data (ADOT shapefiles) are publicly available on Zenodo: `10.5281/zenodo.20836630`

See `Arizona_Case_Study_Stochastic/README.md` for full details.

#### Treatment Cost Analysis

Develops empirical pavement treatment unit cost distributions from ADOT historical
bid tab records. All bid prices are first inflation-adjusted to **2025 Q1 real
dollars** using the National Highway Construction Cost Index (NHCCI), then analyzed
by treatment type to produce lognormal fits, county- and statewide distribution
plots, and bootstrap-based summary statistics. For multi-component treatments
(e.g., Remove & Replace, Chip Seal, Reconstruction), individual pay items are
combined into composite lane-mile costs using standard ADOT quantity assumptions.
The resulting lognormal parameters (mean, CoV, μ_ln, σ_ln per treatment type) and
the treatment cost correlation matrix serve directly as inputs to the stochastic
network-level model above.

Bid tab data were obtained from BidTabs.NET and cannot be shared publicly due to
licensing. The NHCCI deflator file (`NHCCI_20251220.csv`) is included in the folder.

See `Arizona_Treatment_Cost_Analysis/README.md` for full details.

---

### Minnesota Case Study

Evaluates agency and user costs for Minnesota's state highway network over a
10-year analysis period (2025–2034) using MnDOT's Capital Highway Investment
Plan (CHIP) dataset. IRI values are derived from MnDOT's projected Riding
Quality Index (RQI) values. The notebook reproduces Figs. 3–7 in the paper:

- RQI and SR trends (Fig. 3)
- Weighted average IRI trend (Fig. 4)
- Treatment distribution by year (Fig. 5)
- User costs by vehicle type (Fig. 6)
- Agency costs from MnDOT CHIP program (Fig. 7)

The MnDOT dataset is restricted and cannot be shared publicly. A dummy dataset
(`mndot_dummy.xlsx`) with the correct column structure is provided. The notebook
is committed with saved outputs so all figures remain visible without re-running.

See `Minnesota Case Study/README.md` for full details.

---

## Data & Code Availability

- **Code (GitHub):** [Pavement Management](https://github.com/Okte-Research-Group/Pavement_Management)
- **Code (Zenodo DOI):** `10.5281/zenodo.20836630` [Zenodo_Code & Data](https://zenodo.org/records/20836630)
- **Arizona data (Zenodo DOI):** `10.5281/zenodo.20836630` [Zenodo_Code & Data](https://zenodo.org/records/20836630)
- **Minnesota data:** Restricted — provided by MnDOT for research purposes only.
- **Bid tab data (Arizona treatment costs):** Restricted — obtained from BidTabs.NET. Contact the corresponding author for access inquiries.
- **NHCCI construction cost index:** Publicly available from FHWA (included as `Arizona_Treatment_Cost_Analysis/NHCCI_20251220.csv`).

---

## Installation

Python 3.10–3.12 is recommended. Each case study folder has its own
`requirements.txt` (Arizona) or inline install instructions (Minnesota).

**Arizona (Deterministic):**
```bash
cd "Arizona Case Study - Deterministic LCCA"
pip install -r requirements.txt
```

**Arizona (Stochastic):**
```bash
cd Arizona_Case_Study_Stochastic
pip install -r requirements.txt
```

**Arizona (Treatment Cost Analysis):**
```bash
pip install pandas numpy matplotlib scipy
```

**Minnesota:**
```bash
pip install pandas numpy matplotlib seaborn openpyxl
```

---

## How to Cite

If you use this code or data, please cite both the manuscript and the archived release

---

## Acknowledgments

This work is supported by the Federal Highway Administration (FHWA) under the
project *Cost-Efficient Network-Level Pavement Management Framework for Flexible
Pavement Preservation and Maintenance*. The authors acknowledge the FHWA team
led by Sivaneswaran Nadarajah, and the support of the Arizona Department of
Transportation (ADOT) and the Minnesota Department of Transportation (MnDOT).

---

## License

`Arizona case study material is licensed under the MIT License.`
