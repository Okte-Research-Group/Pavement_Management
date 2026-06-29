# Network-Level Pavement Probabilistic Management — Arizona (ADOT) Stochastic Case Study

This repository contains the **stochastic**, network-level pavement management (PMS) model for the **Arizona case study**, developed as part of the FHWA EAR project:

> **Cost-Efficient Network-Level Pavement Management Framework for Flexible Pavement Preservation and Maintenance**

The code reproduces the Arizona stochastic analysis: it loads the ADOT pavement network, projects pavement deterioration over a 10-year horizon under **Monte Carlo uncertainty**, applies the ADOT treatment decision tree, allocates a constrained annual budget under three prioritization strategies, and reports both **agency costs** and **road-user costs** (excess fuel consumption from pavement roughness), together with their probability distributions, for each strategy.

The companion **deterministic** version of this case study is available at:
[Arizona Case Study — Deterministic](https://github.com/Okte-Research-Group/Pavement_Management/tree/main/Arizona%20Case%20Study%20-%20Deterministic%20LCCA)

---

## What the model does

A single Jupyter notebook, `Arizona_Case_Study_Stochastic.ipynb`, runs the full workflow end to end:

1. **Load & preprocess** the 0.1-mile base-segment network and the merged 5-mile decision network, and build the base-to-merged mapping.
2. **Compute structural inputs** — 20-year ESAL, pavement family, and the structural/seasonal variability factor.
3. **Project performance** under uncertainty — IRI, rutting, and cracking deterioration models (ADOT coefficients) with multiplicative noise on the annual increment, sampled once per Monte Carlo realization.
4. **Sample treatment unit costs** once per Monte Carlo realization from a lognormal distribution — either independently or from a correlated joint distribution across treatment types.
5. **Select treatments** with the ADOT decision tree (18 treatment types keyed to condition, functional class, AADT, rehab history, and scheduling state).
6. **Allocate a constrained annual budget** under three strategies.
7. **Estimate costs** — agency cost (sampled unit cost × treated lane-miles) and user cost (excess fuel consumption relative to a baseline IRI, by vehicle class).
8. **Aggregate Monte Carlo results** — compute mean trajectories and percentile bands across all realizations.
9. **Compare strategies** and export results to CSV.

### Prioritization strategies

| Strategy | Function name | Ranking rule |
|---|---|---|
| **Worst-first** | `simulate_network_mc_worst_first` | Most-distressed segments first, by a composite IRI/cracking/rutting index |
| **Preservation** | `simulate_network_mc_benefit_cost` | Benefit/cost ratio, where benefit = area between the do-nothing and post-treatment performance curves |
| **Traffic-Weighted Preservation** | `simulate_network_mc_benefit_cost_aadt` | Same benefit/cost, additionally weighted by AADT² to favor high-traffic corridors |

### Sources of uncertainty

| Source | How it is modeled |
|---|---|
| **Treatment unit cost** | Sampled once per MC run from a lognormal distribution; optionally correlated across treatment types via a Cholesky decomposition of the empirical cost correlation matrix |
| **Deterioration rate** | Multiplicative noise on the annual IRI, rutting, and cracking increment: `TP_{t+1} = TP_t + ΔTP × (1 + CoV × z)`, where `z` is drawn once per MC run per metric |

---

## Repository / data layout

The notebook resolves all paths relative to its own directory. After cloning, your folder should look like this:

```
code-Segmentation-5miles/
├── Arizona_Case_Study_Stochastic.ipynb
├── paired_rr_ac_fr_correlation_matrix.csv   # treatment cost correlation matrix
├── requirements.txt
├── README.md
└── adot_data 2/
    └── gis/
        ├── before_merge_0p1miles_arizona/
        │   ├── Arizona.shp
        │   ├── Arizona.dbf
        │   ├── Arizona.shx
        │   ├── Arizona.prj
        │   └── ...                          # remaining shapefile sidecar files
        └── merged_5miles_arizona/
            ├── Merged_Arizona.shp
            ├── Merged_Arizona.dbf
            ├── Merged_Arizona.shx
            ├── Merged_Arizona.prj
            └── ...
```

> **Note on shapefiles:** a shapefile is not a single file. Keep every sidecar file (`.shp`, `.shx`, `.dbf`, `.prj`, and any others) together in the same directory, or GeoPandas will fail to read it.

### Expected input columns

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

Python 3.13.2 or later is recommended.

```bash
# (optional) create an isolated environment
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate

pip install -r requirements.txt
```

---

## Running

1. Place the `adot_data 2/` folder next to the notebook (see layout above).
2. Launch Jupyter and open the notebook:
   ```
   jupyter lab        # or: jupyter notebook
   ```
3. **Edit the USER CONFIGURATION cell** (the first code cell) to choose strategies, budget, and uncertainty parameters.
4. **Run all cells, top to bottom.** The comparison section requires outputs from all three strategy blocks, so do not skip any run block if you want the comparison figures and combined CSV exports.

### Configuration

All user-tunable settings live in the first **USER CONFIGURATION** cell:

| Setting | Default | Description |
|---|---|---|
| `RUN_WORST_FIRST`, `RUN_BENEFIT_COST`, `RUN_BENEFIT_COST_AADT` | `1` | Toggle each strategy on (`1`) / off (`0`) |
| `USE_COST_CORRELATION` | `1` | `1` = correlated lognormal cost sampling; `0` = independent sampling |
| `USER_COST_IRI_BASE` | `60` | IRI reference for a road in good condition (in/mi) |
| `COV_IRI` | `0.0001` | Coefficient of variation for IRI deterioration noise |
| `COV_RUTTING` | `0.0001` | Coefficient of variation for rutting deterioration noise |
| `COV_FATIGUE` | `0.0001` | Coefficient of variation for fatigue cracking deterioration noise |
| `N_MC` | `400` | Monte Carlo iterations for the single-budget run |
| `SEED` | `123` | Random seed for reproducibility |
| `YEARS` | `10` | Analysis horizon (years) |
| `BUDGET` | `440_000_000` | Annual agency budget for the single-budget run ($) |
| `BUDGETS_MULTI` | `[20M, 100M, 300M, 500M, 800M]` | Budget levels for the multi-budget sensitivity sweep ($) |
| `N_MC_MULTI` | `400` | Monte Carlo iterations per budget level in the multi-budget run |
| `COMPOSITE_*` | — | Scale factors and weights of the composite condition index |
| `TIMING_N_MC_VALUES` | `[1, 50, 250, 500, 750]` | Sample sizes for the runtime-scaling experiment |
| `SAVE_SEGMENT_RESULTS` | `0` | `1` = export per-run Parquet files with segment-level condition data |
| `SEGMENT_RESULTS_DIR` | `"segment_results"` | Output directory for Parquet files |
| `OUTPUT_DIR` | auto | Set to `Correlation_Results` or `Without_Correlation_Results` based on `USE_COST_CORRELATION` |

The `$440M` default budget and the 10-year horizon reflect the average annual pavement-preservation funding from the Arizona TAMP, in constant 2024 dollars.

---

## Outputs

Running the notebook produces, in the `OUTPUT_DIR` folder (created automatically):

**Single-budget results** (one row per Monte Carlo realization per year):
- `single_budget_simulation_worstfirst_cov{COV_IRI}.csv`
- `single_budget_simulation_benefit_cov{COV_IRI}.csv`
- `single_budget_simulation_benefit_aadt_cov{COV_IRI}.csv`

**Multi-budget results** (includes a `budget` column):
- `multi_budget_simulation_results_worstfirst_cov{COV_IRI}.csv`
- `multi_budget_simulation_results_benefit_cov{COV_IRI}.csv`
- `multi_budget_simulation_results_benefit_aadt_cov{COV_IRI}.csv`

Each CSV contains the columns: `simulation`, `year`, `weighted_avg_iri`, `weighted_avg_rutting`, `weighted_avg_cracking`, `weighted_avg_composite`, `user_cost` (and `budget` for multi-budget files).

**Inline figures** — convergence diagnostics, condition fan plots (mean ± P10–P90), user-cost trajectories, final-year violin plots, and multi-budget sensitivity curves.

---

## Method notes

- **Two resolutions.** Treatment selection, prioritization, and budget allocation happen at the merged 5-mile **decision** segment level, while deterioration models and treatment resets are applied at the 0.1-mile **reporting-unit** level and aggregated back with lane-mile weighting.
- **Monte Carlo design.** Each realization draws one global performance-shock scalar per metric (`z_iri`, `z_rut`, `z_crack`) and one unit cost per treatment type. These are held fixed across all segments and years within a realization, so uncertainty reflects systematic network-wide variation rather than independent segment noise.
- **Cost correlation.** When `USE_COST_CORRELATION = 1`, the R&R treatment costs are sampled jointly from a multivariate lognormal distribution whose correlation matrix is loaded from `paired_rr_ac_fr_correlation_matrix.csv`. Treatments not in the correlation matrix are sampled independently.
- **User cost** is the excess fuel consumption (energy above a baseline-IRI reference) monetized with 2024 West-Coast fuel prices, split across four vehicle classes (passenger, small/medium/large trucks). A mid-year IRI is used for each year's user-cost calculation.
- **Budget allocation** uses a skip-and-continue greedy scan: projects are funded in priority order if they fit the remaining budget; an unaffordable project is skipped rather than blocking lower-priority projects that still fit.

---

## Data & code availability

- **Code:** archived on GitHub and Zenodo.
  - GitHub: [Pavement Management](https://github.com/Okte-Research-Group/Pavement_Management)
- **Data:** the ADOT pavement network shapefiles are archived on Zenodo.
  - Zenodo (data DOI): `10.5281/zenodo.20836630` — [Zenodo Code & Data](https://zenodo.org/records/20836630)

The input data are derived from the 2022 ADOT Highway Performance Monitoring System (HPMS) pavement inventory.

---

## Acknowledgments

This work is part of the FHWA project *Cost-Efficient Network-Level Pavement Management Framework for Flexible Pavement Preservation and Maintenance*. The authors acknowledge the FHWA team led by Sivaneswaran Nadarajah, and the support of the Arizona Department of Transportation.

