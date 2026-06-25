# Network-Level Pavement Life-Cycle Planning — Arizona (ADOT) Deterministic Case Study

This repository contains the deterministic, network-level pavement
life-cycle planning (LCP) model for the **Arizona case study** described in the
manuscript:

> **Impact of Integrating Use Stage into Network-Level Life-Cycle Planning**
> S. Uhde Yildirim, S. Mostatab, M. Zeigham, E. Okte (corresponding author),
> E. Tseng, I. L. Al-Qadi, and H. Ozer.

The code reproduces the Arizona analysis: it loads the ADOT pavement network,
projects pavement deterioration over a 10-year horizon, applies the ADOT
treatment decision tree, allocates a constrained annual budget under three
prioritization strategies, and reports both **agency costs** and **road-user
costs** (excess fuel consumption from pavement roughness) for each strategy.

> The companion **Minnesota** case study in the manuscript is based on the
> MnDOT CHIP database and is **not** part of this code release.

---

## What the model does

A single Jupyter notebook, `Arizona_Case_Study_Deterministic_LCCA.ipynb`,
runs the full workflow end to end:

1. **Load & preprocess** the 0.1-mile base-segment network and the merged
   5-mile decision network, and build the base-to-merged mapping.
2. **Compute structural inputs** — 20-year ESAL, pavement family, and the
   structural/seasonal variability factor.
3. **Project performance** — deterministic IRI, rutting, and cracking
   deterioration models (ADOT coefficients).
4. **Select treatments** with the ADOT decision tree (18 treatment types keyed
   to condition, functional class, AADT, rehab history, and scheduling state).
5. **Allocate a constrained annual budget** under three strategies.
6. **Estimate costs** — agency cost (unit cost × treated lane-miles) and user
   cost (excess fuel consumption relative to a baseline IRI, by vehicle class).
7. **Compare strategies** and export results to CSV and per-segment Parquet.

### Prioritization strategies

| Strategy | Notebook name | Ranking rule |
|---|---|---|
| **Worst-first** | `_simulate_years_worst_first` | Most-distressed segments first, by a composite IRI/cracking/rutting index |
| **Preservation** | `_simulate_years_benefit_cost` | Benefit/cost ratio, where benefit = area between the do-nothing and post-treatment performance curves |
| **Traffic-Weighted Preservation** | `_simulate_years_benefit_cost_aadt` | Same benefit/cost, additionally weighted by AADT² to favor high-traffic corridors |

The run is **fully deterministic**: a fixed random seed sets the initial
pavement-management state (pavement age, rehab count, scheduling year) once, and
no sampling occurs afterward, so results are exactly reproducible.

---

## Repository / data layout

The notebook is written so that **if the data folder sits next to the
notebook, it runs as-is** — it resolves all paths relative to the current
working directory (`Path.cwd()`). After downloading and extracting the data
from Zenodo, your folder should look like this:

```
project_folder/
├── Arizona_Case_Study_Deterministic_LCCA.ipynb
├── requirements.txt
├── README.md
└── adot_data/
    └── gis/
        ├── before_merge_0p1miles_arizona/
        │   ├── Arizona.shp
        │   ├── Arizona.dbf
        │   ├── Arizona.shx
        │   ├── Arizona.prj
        │   └── ...                       # remaining shapefile sidecar files
        └── merged_5miles_arizona/
            ├── Merged_Arizona.shp
            ├── Merged_Arizona.dbf
            ├── Merged_Arizona.shx
            ├── Merged_Arizona.prj
            └── ...
```

The notebook checks that both shapefiles exist and stops with a clear message if
the `adot_data/gis/...` folders are missing, so place the Zenodo data before
running.

> **Note on shapefiles:** a shapefile is not a single file. Keep every sidecar
> file (`.shp`, `.shx`, `.dbf`, `.prj`, and any others) together in the same
> directory, or GeoPandas will fail to read it.

### Expected input columns

The two shapefiles are expected to carry the following key fields (column names
follow the 10-character shapefile/DBF convention).

**Base network — `Arizona.shp`:**

| Field | Meaning |
|---|---|
| `FID_N` | Unique base-segment ID (links to the merged network's `Original_I`) |
| `PAVEMENT_T` | Surface type (`AC`, `Other`, `JPCP`, `CRCP`, …) |
| `AvgIRI`, `HPMS_Crack`, `Rutting` | Initial condition: IRI (in/mi), HPMS cracking (%), rut depth |
| `IRI_Rating`, `Cracking_R`, `Rutting_Ra` | Condition ratings (1 = Good, 2 = Fair, 3 = Poor) |
| `FromMeasur`, `ToMeasure` | Begin/end milepost (segment length = difference) |
| `Number_of_` | Number of through lanes (per direction) |
| `Functional` | ADOT functional-class code |
| `AADT` | Annual average daily traffic |
| `AVERAGE_SP` | Average operating speed (used in the user-cost model) |
| `Length_Mil`, `TARGET_FID` | Segment length (mi) and target ID |

**Merged decision network — `Merged_Arizona.shp`:**

| Field | Meaning |
|---|---|
| `Original_I` | List of base `FID_N` values grouped into the 5-mile management segment |
| `PAVEMENT_T` | Surface type of the merged segment |
| `Length_Mil` | Merged-segment length (mi) |

---

## Installation

Python 3.10–3.12 is recommended.

```bash
# (optional) create an isolated environment
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate

pip install -r requirements.txt
```

> **Important:** the model requires **NumPy ≥ 2.0** because the benefit
> (Area-Under-Curve) calculations use `numpy.trapezoid()`, which does not exist
> in NumPy 1.x. Reading the shapefiles requires GeoPandas with a vector engine
> (`pyogrio`, included in `requirements.txt`).

---

## Running

1. Place the `adot_data/` folder next to the notebook (see layout above).
2. Launch Jupyter and open the notebook:
   ```bash
   jupyter lab        # or: jupyter notebook
   ```
3. **Run all cells, top to bottom.** The comparison section needs the outputs
   of all three strategy blocks, so do not skip any of the three run blocks if
   you want the comparison figures and the combined CSV exports.

### Configuration

All user-tunable settings live in the first **USER CONFIGURATION** cell:

| Setting | Default | Description |
|---|---|---|
| `RUN_WORST_FIRST`, `RUN_BENEFIT_COST`, `RUN_BENEFIT_COST_AADT` | `1` | Toggle each strategy on (`1`) / off (`0`) |
| `SEED` | `123` | Fixes the one-time draw of the initial PMS state; makes the run reproducible |
| `YEARS` | `10` | Analysis horizon (years) |
| `BUDGET` | `440_000_000` | Annual budget for the single-budget run (constant 2024 $) |
| `BUDGETS_MULTI` | `[20M, 100M, 300M, 500M, 800M]` | Budget levels for the sensitivity sweep |
| `COMPOSITE_*` | — | Scale factors and weights of the composite condition index (matches the worst-first ranking) |
| `SAVE_SEGMENT_RESULTS` | `1` | Write per-segment, per-year condition data to Parquet |
| `SEGMENT_RESULTS_DIR` | `"segment_results"` | Output directory for the Parquet files |
| `SEGMENT_RESULTS_COMPRESSION` | `"snappy"` | Parquet compression codec |
| `TREATMENT_TO_CATEGORY` | — | Maps each treatment to a budget category (Preservation / Major Projects / Reconstruction) |

The `$440M` default budget and the 10-year horizon match the manuscript's
Arizona case study (the average annual pavement-preservation funding from the
Arizona TAMP, in constant 2024 dollars).

---

## Outputs

Running the notebook produces, in the working directory:

- **`single_budget_simulation_deterministic.csv`** — per-strategy, per-year
  lane-mile-weighted average IRI, rutting, cracking, composite index, and user
  cost at the single budget.
- **`multi_budget_simulation_results_deterministic.csv`** — the same metrics
  across every budget in `BUDGETS_MULTI`.
- **`segment_results/`** (when `SAVE_SEGMENT_RESULTS = 1`) — Parquet files with
  per-segment yearly IRI / cracking / rutting, partitioned as
  `segment_results/strategy=<name>/budget=<value>/simulation=0000.parquet`,
  plus a `segments.parquet` metadata table.
- **Inline figures** — condition trajectories, user-cost curves, treated-length
  by category, user cost by vehicle type, and the budget-sensitivity plots that
  correspond to the manuscript's Arizona figures.

---

## Method notes

- **Two resolutions.** Treatment selection, prioritization, and budget
  allocation happen at the merged 5-mile **decision** segment level, while
  deterioration models and treatment resets are applied at the 0.1-mile
  **reporting-unit** level and aggregated back with lane-mile weighting.
- **User cost** is the excess fuel consumption (energy above a baseline-IRI
  reference) monetized with 2024 West-Coast fuel prices, split across four
  vehicle classes (passenger, small/medium/large trucks). A mid-year IRI is used
  for each year's user-cost calculation.
- **Budget allocation** uses a skip-and-continue greedy scan: projects are funded
  in priority order if they fit the remaining budget; an unaffordable project is
  skipped rather than blocking lower-priority projects that still fit.

---

## Data & code availability

- **Code:** archived on GitHub and Zenodo.
  - GitHub: `<ADD GITHUB URL>`
  - Zenodo (code DOI): `<ADD ZENODO DOI>`
- **Data:** the ADOT pavement network shapefiles are archived on Zenodo.
  - Zenodo (data DOI): `<ADD ZENODO DATA DOI>`

The input data are derived from the 2022 ADOT Highway Performance Monitoring
System (HPMS) pavement inventory.

---

## How to cite

If you use this code or data, please cite both the manuscript and the archived
release:

```bibtex
@article{UhdeYildirim_LCP_Arizona,
  title   = {Impact of Integrating Use Stage into Network-Level Life-Cycle Planning},
  author  = {Uhde Yildirim, Semiha and Mostatab, Seyedehzahra and Zeigham, Mohammad
             and Okte, Egemen and Tseng, Ester and Al-Qadi, Imad L. and Ozer, Hasan},
  year    = {<YEAR>},
  journal = {<JOURNAL / PROCEEDINGS>},
  doi     = {<ARTICLE DOI>}
}

@software{ArizonaLCCA_code,
  title     = {Network-Level Pavement Life-Cycle Planning — Arizona Deterministic Case Study},
  author    = {Uhde Yildirim, Semiha and Mostatab, Seyedehzahra and Zeigham, Mohammad
               and Okte, Egemen and Tseng, Ester and Al-Qadi, Imad L. and Ozer, Hasan},
  year      = {<YEAR>},
  publisher = {Zenodo},
  doi       = {<ZENODO CODE DOI>}
}
```

---

## Acknowledgments

This work is part of the FHWA project *Cost-Efficient Network-Level Pavement
Management Framework for Flexible Pavement Preservation and Maintenance*. The
authors acknowledge the FHWA team led by Sivaneswaran Nadarajah, and the support
of the Arizona and Minnesota Departments of Transportation.

## License

`<ADD LICENSE — e.g., MIT for code, CC BY 4.0 for data>`
