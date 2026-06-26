# Minnesota Case Study – User Cost and IRI Analysis

This folder contains the code for the Minnesota case study presented in:

> **"Impact of Integrating Use Stage into Network Level Life Cycle Planning"**  
The notebook evaluates **agency costs** and **road-user costs** for Minnesota's
state highway network over a 10-year analysis period (2025–2034), using
MnDOT's Capital Highway Investment Plan (CHIP) dataset.

---

## What the model does

A single Jupyter notebook, `minnesota_case_study.ipynb`, runs the full workflow:

1. **Load & preprocess** the MnDOT CHIP dataset, filtering to flexible pavement
   types (BAB, BFD, BOB, BOC).
2. **Convert RQI to IRI** using the formula:
   ```
   IRI = ((5.697 - RQI) / 0.264)²
   ```
   applied for each year from 2024 to 2035.
3. **Compute weighted average IRI** using lane-mile weights across the network.
4. **Assign speeds** by functional class (Local: 30 mph → Interstate: 70 mph).
5. **Estimate user costs** (excess fuel consumption) using the Roughness Speed
   Impact (RSI) model (Ziyadi et al., 2018, Equation 7), for four vehicle
   classes with a baseline IRI of 20 in/mi (0.32 m/km).
6. **Compute agency costs** from the MnDOT Data Program sheet, filtered to
   12 flexible pavement treatment activities.
7. **Produce five figures** corresponding to Figs. 3–7 in the paper.

---

## Figures produced

| Figure | Section | Description |
|--------|---------|-------------|
| Fig. 3 | §5 | Lane-mile weighted RQI and SR trends (2025–2034) |
| Fig. 4 | §4 | Lane-mile weighted average IRI trend in m/km (2025–2034) |
| Fig. 5 | §8 | Treatment type distribution by year (% share, 2025–2034) |
| Fig. 6 | §11 | Annual user costs by vehicle type (Million $, 2025–2034) |
| Fig. 7 | §12 | Annual agency costs from MnDOT CHIP program (Million $, 2025–2034) |

---

## User cost model

User costs represent **excess fuel consumption (EFC)** above a baseline IRI of
20 in/mi. The RSI model computes energy consumed per vehicle mile traveled as a
function of vehicle speed and pavement roughness (IRI):

```
E = [(p / Speed) + (ka × IRI + da) + (b × Speed) + ((kc × IRI + dc) × Speed²)] / 1000
```

Four vehicle classes are considered:

| Vehicle Class | Fleet Share | Fuel Type | Price ($/gal) |
|---|---|---|---|
| Passenger Vehicle | 70% | Gasoline | $2.992 |
| Small Truck | 10% | Gasoline | $2.992 |
| Medium Truck | 15% | Diesel | $3.456 |
| Large Truck | 5% | Diesel | $3.456 |

Fuel prices are based on 2025 values. Traffic growth rate is assumed to be 0%.

---

## Speed limits by functional class

| Functional Class | Speed (mph) |
|---|---|
| Local | 30 |
| Major Collector | 55 |
| Minor Collector | 55 |
| Minor Arterial | 55 |
| Principal Arterial | 55 |
| Principal Arterial – Freeway | 65 |
| Interstate | 70 |

---

## Agency cost treatments included

The following 12 flexible pavement activities from the MnDOT Data Program sheet
are included in the agency cost analysis:

`BAB_Constr_Urban`, `BAB_Constr_Rural`, `Medium_Mill_Overlay`,
`Thin_Mill_Overlay`, `Thick_Mill_Overlay`, `Thick_Overlay`, `Thin_Overlay`,
`Medium_Overlay`, `Micro_Surfacing`, `Reclaim_Overlay`, `Nova_Chip_UTBWC`,
`CIR & Medium OL`

Concrete treatments (Whitetop, CD Construction, Unbonded Overlay) are excluded.

---

## Repository contents

| File | Description |
|------|-------------|
| `minnesota_case_study.ipynb` | Main analysis notebook (outputs saved) |
| `mndot_dummy.xlsx` | Dummy dataset with the correct structure for reproducibility |
| `README.md` | This file |

---

## Data availability

The MnDOT CHIP dataset was provided for research purposes under a data sharing
agreement and **cannot be publicly shared**. A dummy dataset (`mndot_dummy.xlsx`)
with the correct column structure is provided so the notebook can be inspected
and the code can be adapted to other datasets.

The notebook is committed with saved cell outputs so that all figures (Figs. 3–7)
remain visible on GitHub without re-running the code.

### Expected input columns

**CHIP sheet:**

| Column | Description |
|--------|-------------|
| `Pvmt_Type` | Pavement type (BAB, BFD, BOB, BOC for flexible) |
| `RQI_{year}` | Riding Quality Index for each year (2024–2035) |
| `SR_{year}` | Structural Rating for each year (2025–2034) |
| `Funct_Class` | Functional classification |
| `AADT` | Annual average daily traffic |
| `Lane_Miles` | Lane miles per segment |
| `No_of_Lanes` | Number of lanes |
| `Lengt_miles` | Segment length (miles) |
| `Planned_Work_Type` | Planned treatment type |
| `Planned_Work_Year` | Year the treatment is planned |

**Data Program sheet:**

| Column | Description |
|--------|-------------|
| `Activity` | Treatment activity name |
| `Year` | Treatment year |
| `Cost` | Treatment cost ($) |

---

## Installation

Python 3.10–3.12 is recommended.

```bash
pip install pandas numpy matplotlib seaborn openpyxl
```

---

## Running the notebook

1. Place `mndot_dummy.xlsx` (or your own data file) in the same folder as the notebook.
2. Update `file_path` in Section 1 and Section 12 to point to your data file.
3. Launch Jupyter and open the notebook:
   ```bash
   jupyter lab
   ```
4. Run all cells top to bottom.

> **Note:** The notebook was originally developed in Google Colab. If running
> locally, remove the `from google.colab import drive` and `drive.mount()`
> lines from Section 1.

---

## Data & code availability

- **Code:** archived on GitHub and Zenodo.
  - GitHub: `<ADD GITHUB URL>`
  - Zenodo (code DOI): `<ADD ZENODO DOI>`

---

## How to cite

```bibtex
@article{UhdeYildirim_LCP_Minnesota,
  title   = {Impact of Integrating Use Stage into Network-Level Life-Cycle Planning},
  author  = {Uhde Yildirim, Semiha and Okte, Egemen and others},
  year    = {<YEAR>},
  journal = {<JOURNAL / PROCEEDINGS>},
  doi     = {<ARTICLE DOI>}
}
```

---

## Acknowledgments

This work is part of the FHWA project *Cost-Efficient Network-Level Pavement
Management Framework for Flexible Pavement Preservation and Maintenance*.
The authors acknowledge the FHWA team led by Sivaneswaran Nadarajah, and the
support of the Minnesota Department of Transportation (MnDOT).

## License

`<ADD LICENSE>`
