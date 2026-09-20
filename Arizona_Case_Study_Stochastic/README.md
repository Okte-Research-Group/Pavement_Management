# Network-Level Pavement Probabilistic Management — Arizona (ADOT) Stochastic Case Study

This repository contains the **stochastic**, network-level pavement management (PMS) model for the **Arizona case study**, developed as part of the FHWA EAR project:

> **Cost-Efficient Network-Level Pavement Management Framework for Flexible Pavement Preservation and Maintenance**

The code reproduces the Arizona stochastic analysis: it loads the ADOT pavement network, projects pavement deterioration over a 10-year horizon under **Monte Carlo uncertainty**, applies the ADOT treatment decision tree, allocates a constrained annual budget under three prioritization strategies, and reports both **agency costs** and **road-user costs** (excess fuel consumption from pavement roughness), together with their probability distributions, for each strategy. It also reports network condition **reliability** (Good/Fair/Poor by functional category) and **State of Good Repair (SOGR)** failure probability by functional group.

This case study, together with the companion Treatment Cost Analysis and the
post-processing notebook below, supports the manuscript:

> **"Probabilistic Pavement Management Using Bid-Based Cost Distributions"**  
> M. Zeigham, S. Mostatab, S. U. Yildirim, E. Okte (corresponding author),  
> E. Tseng, H. Ozer, and I. Al-Qadi.

The companion **deterministic** version of this case study is available at:
[Arizona Case Study — Deterministic](https://github.com/Okte-Research-Group/Pavement_Management/tree/main/Arizona%20Case%20Study%20-%20Deterministic%20LCCA)

---

## What the model does

The main notebook, `Arizona_Case_Study_Stochastic.ipynb`, runs the full simulation workflow end to end:

1. **Load & preprocess** the 0.1-mile base-segment network and the merged 5-mile decision network, and build the base-to-merged mapping.
2. **Compute structural inputs** — 20-year ESAL, pavement family, and the structural/seasonal variability factor.
3. **Project performance** under uncertainty — IRI, rutting, and cracking deterioration models (ADOT coefficients) with a calibrated, strictly positive lower-truncated-normal multiplier applied to the annual increment, sampled once per Monte Carlo realization.
4. **Sample treatment unit costs** once per Monte Carlo realization from a lognormal distribution — either independently or from a correlated joint distribution across treatment types.
5. **Select treatments** with the ADOT decision tree (18 treatment types keyed to condition, functional class, AADT, rehab history, and scheduling state).
6. **Allocate a constrained annual budget** under three strategies.
7. **Estimate costs** — agency cost (sampled unit cost × treated lane-miles) and user cost (excess fuel consumption relative to a baseline IRI, by vehicle class).
8. **Aggregate Monte Carlo results** — compute mean trajectories and percentile bands across all realizations.
9. **Compare strategies** and export results to CSV.
10. **Report reliability** — lane-mile weighted Good/Fair/Poor condition distribution by functional category, and State of Good Repair (SOGR) failure probability and mean SOGR by functional group and year (see [Reliability & SOGR analysis](#reliability--sogr-analysis) below).

A companion notebook, `Arizona_Case_Study_Stochastic_Post_Processing.ipynb`, consumes this
notebook's CSV/Parquet outputs to produce the manuscript-ready figures (strategy comparisons,
CCDF curves, pairwise outperformance heatmaps, multi-budget and CoV sensitivity curves) in SI
units. Run the main notebook first, then the post-processing notebook.

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
| **Deterioration rate** | A calibrated multiplier applied to the annual IRI, rutting, and cracking increment: `TP_{t+1} = TP_t + ΔTP × multiplier`, where `multiplier` is drawn once per MC run per metric from a lower-truncated-normal distribution calibrated so that `mean(multiplier) = 1` and `CoV(multiplier) = COV_IRI`/`COV_RUTTING`/`COV_FATIGUE`. The lower truncation at 0 keeps the multiplier strictly positive (pavement condition cannot spontaneously improve absent a treatment), unlike a plain normal shock `(1 + CoV × z)`, which can go negative at high CoV |

---

## Reliability & SOGR analysis

In addition to the Monte Carlo cost/condition simulation, the notebook includes two
condition-reliability analyses, each self-contained (own imports, own data load):

- **Network condition reliability** — lane-mile weighted Good/Fair/Poor condition distribution
  by functional category (Freeways & Interstate, Other Arterials, Collectors & Local), using
  ADOT's `IRI_Rating`, `Cracking_R`, and `Rutting_Ra` fields on the baseline 0.1-mile network.
- **State of Good Repair (SOGR)** — probability of failure and mean SOGR by year, for four
  functional groups, computed from the 150 Monte Carlo simulations already generated per
  strategy at the `$440,000,000` budget level (`segment_results/strategy=.../budget=440000000/`):

  | Group | Functional codes | Fail rule |
  |---|---|---|
  | Freeways & Interstate | 1, 3, 11, 12 | SOGR >= 2% |
  | Other Arterials | 2, 6, 14, 16 | SOGR >= 7% |
  | Collectors | 7, 8, 17, 18 | SOGR >= 7% |
  | Rural (Local) | 9, 19 | Split by AADT: fails if High-AADT (>400) sub-SOGR >= 7% **OR** Low-AADT (<=400) sub-SOGR >= 15% |

  The SOGR analysis requires an `AADT` column joined onto `segments_cov0.15.parquet`, added by
  a one-time data-prep cell earlier in the notebook (run once; safe to skip on subsequent runs).

---

## Repository / data layout

The notebook resolves all paths relative to its own directory. After cloning, your folder should look like this:

```
code-Segmentation-5miles/
├── Arizona_Case_Study_Stochastic.ipynb
├── Arizona_Case_Study_Stochastic_Post_Processing.ipynb
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
- **Monte Carlo design.** Each realization draws one calibrated performance-uncertainty multiplier per metric (IRI, rutting, cracking; mean 1, CoV = `COV_IRI`/`COV_RUTTING`/`COV_FATIGUE`) and one unit cost per treatment type. These are held fixed across all segments and years within a realization, so uncertainty reflects systematic network-wide variation rather than independent segment noise.
- **Cost correlation.** When `USE_COST_CORRELATION = 1`, the R&R treatment costs are sampled jointly from a multivariate lognormal distribution whose correlation matrix is loaded from `paired_rr_ac_fr_correlation_matrix.csv`; if that file is missing, the notebook raises an error rather than silently falling back to independent sampling. Treatments not in the correlation matrix are sampled independently. Set `USE_COST_CORRELATION = 0` to intentionally run with independent cost sampling.
- **User cost** is the excess fuel consumption (energy above a baseline-IRI reference) monetized with 2024 West-Coast fuel prices, split across four vehicle classes (passenger, small/medium/large trucks). A mid-year IRI is used for each year's user-cost calculation.
- **Budget allocation** uses a skip-and-continue greedy scan: projects are funded in priority order if they fit the remaining budget; an unaffordable project is skipped rather than blocking lower-priority projects that still fit.

---

## Data & code availability

- **Code:** archived on GitHub and Zenodo.
  - GitHub: [Pavement Management](https://github.com/Okte-Research-Group/Pavement_Management)
  - Zenodo (code & data DOI): `10.5281/zenodo.22864634` — [Probabilistic Pavement Management Using Bid-Based Cost Distributions](https://doi.org/10.5281/zenodo.22864634)
- **Data:** the ADOT pavement network shapefiles and treatment cost distributions used by this notebook are archived in the Zenodo record above.

The input data are derived from the 2022 ADOT Highway Performance Monitoring System (HPMS) pavement inventory.

---

## How to cite

If you use this code or data, please cite the manuscript and the archived Zenodo dataset release.

```bibtex
@article{ProbabilisticPMS,
  title   = {Probabilistic Pavement Management Using Bid-Based Cost Distributions},
  author  = {Zeigham, Mohammad and
             Mostatab, Seyedehzahra and
             Yildirim, Semiha Uhde and
             Okte, Egemen and
             Tseng, Ester and
             Ozer, Hasan and
             Al-Qadi, Imad},
  year    = {TBD},
  journal = {TBD},
  doi     = {TBD}
}

@dataset{ArizonaDatasetProbabilistic,
  title     = {Probabilistic Pavement Management Using Bid-Based Cost Distributions},
  author    = {Zeigham, Mohammad and
               Mostatab, Seyedehzahra and
               Okte, Egemen},
  year      = {2026},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.22864634}
}
```

GitHub repository: [Pavement Management](https://github.com/Okte-Research-Group/Pavement_Management)

---

## Acknowledgments

This work is part of the FHWA project *Cost-Efficient Network-Level Pavement Management Framework for Flexible Pavement Preservation and Maintenance*. The authors acknowledge the FHWA team led by Sivaneswaran Nadarajah, and the support of the Arizona Department of Transportation.

