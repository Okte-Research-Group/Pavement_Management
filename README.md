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
Roughness Speed Impact (RSI) model (Ziyadi et al., 2018).

Two case studies are included, each in its own folder:

| Folder | Case Study | Data Source |
|--------|-----------|-------------|
| `Arizona Case Study - Deterministic/` | ADOT statewide highway network | ADOT HPMS 2022 shapefiles (public, on Zenodo) |
| `Minnesota Case Study/` | MnDOT state highway network | MnDOT CHIP dataset (restricted, dummy provided) |

---

## Repository Structure

```
Pavement_Management/
├── Arizona Case Study - Deterministic/
│   ├── Arizona_Case_Study_Deterministic_LCCA.ipynb
│   ├── requirements.txt
│   ├── README.md
│   └── adot_data/
│       └── gis/
│           ├── before_merge_0p1miles_arizona/
│           └── merged_5miles_arizona/
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

Input data (ADOT shapefiles) are publicly available on Zenodo: `<ADD ZENODO DATA DOI>`

See `Arizona Case Study - Deterministic/README.md` for full details.

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

- **Code (GitHub):** `<ADD GITHUB URL>`
- **Code (Zenodo DOI):** `<ADD ZENODO CODE DOI>`
- **Arizona data (Zenodo DOI):** `<ADD ZENODO DATA DOI>`
- **Minnesota data:** Restricted — provided by MnDOT for research purposes only.

---

## Installation

Python 3.10–3.12 is recommended. Each case study folder has its own
`requirements.txt` (Arizona) or inline install instructions (Minnesota).

**Arizona:**
```bash
cd "Arizona Case Study - Deterministic"
pip install -r requirements.txt
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

`<ADD LICENSE>`
