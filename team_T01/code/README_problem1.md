## Problem 1: 5-Week-Ahead Influenza ILI Forecasting (Parts 1-a, 1-b, 1-c)

### 1. Overview

| Item | Description |
|---|---|
| Target | Quantile forecasts of ILI (per 1,000 outpatients, 300-site basis) for 1-5 weeks after the forecast origin |
| Quantiles | 0.025, 0.10, 0.25, 0.50, 0.75, 0.90, 0.975 (95% / 80% / 50% PI) |
| Model | 5 candidates -> select top 2 by sequential-validation WIS -> inverse-WIS-weighted quantile-averaging ensemble (`Ensemble_top2_inverse_WIS`) |
| Candidates | 5 models: `RF_delta`, `ET_delta`, `HGB_delta` (HistGradientBoosting), `Ridge_delta`, `Analog` |
| Inputs | Last 8 weeks of log1p(ILI); 1/2/3/4/8/12-week differences; 3/6/12-week moving averages; 6-week standard deviation; week-of-year sin/cos; log1p detection rates (overall, H1N1pdm09, H3N2, B): current value, value 1 week earlier, and 2-week difference. ARI excluded |
| Prediction intervals | Empirical quantiles of each candidate's sequential-validation log-scale errors (median-centered), back-transformed to the original scale, then ensembled |
| Training exclusions | Training samples whose target week falls in 202010-202220 are excluded; validation origins in 202010-202225 are excluded |
| Reproducibility | `SEED = 2026`, `n_jobs=1`, `OMP/OPENBLAS_NUM_THREADS=1` |

Origins by notebook:

| Notebook | Origins | Task | Data file | Output folder |
|---|---|---|---|---|
| `Problem1_1a_1b.ipynb` | A (2023-37), B (2025-41), C (2024-50), 2026-34 | A, B, C = `1A`; 2026-34 = `1B` | `인플루엔자 2차 학습 데이터.xlsx` | `forecast_results_ensemble_top2_all_origins/` |
| `Problem1_1c.ipynb` | 2026-39 -> 2026-40 to 44 | `1C` | `인플루엔자 2차 학습 데이터_아형 검출률 추가.xlsx` | `forecast_results_task1c_tuned/` |

Differences between 1-a/1-b and 1-c:

| | 1-a / 1-b | 1-c |
|---|---|---|
| Tree settings | 160 trees, `min_samples_leaf=3`, `max_features=0.8` | 100 trees; 3 settings compared, **leaf=1, max_features=1.0** selected (`BEST_LEAF`, `BEST_FEATURES`) |
| Validation | Up to the 32 most recent origins (6-week stride). PI calibration uses leave-one-out errors | 32 most recent origins (6-week stride). The first 8 are used for initial calibration; the remaining 24 are used for ranking. Only errors already observed before each validation origin are used (checked via `validation_audit.csv`) |
| Evaluation | Actual values available, so WIS/RMSE/MAE/coverage are computed | No actuals for weeks 40-44, so final WIS is not computed |

### 2. Folder / Files

```
code/
  Problem1_1a_1b.ipynb
  Problem1_1c.ipynb
```

Place the Excel data files in the **same folder** as the notebooks. Required columns: `년주, 주차, ILI, 전체 검출률(%), A(H1N1)pdm09, A(H3N2), B`.

### 3. Environment

* Python 3.11.x (3.11.9 per the notebook kernel), CPU only
* Packages: `numpy, pandas, scikit-learn, scipy, matplotlib, openpyxl` (`openpyxl` is needed by `pandas.read_excel`)
* Versions: just before submission, save the output of `pip freeze | grep -iE "numpy|pandas|scikit-learn|scipy|matplotlib|openpyxl"` as `requirements.txt` and attach it **(TODO: not yet recorded)**

### 4. Execution Order

#### 4-1. 1-a / 1-b (`Problem1_1a_1b.ipynb`)

1. Run the first code cell (function and constant definitions).
2. In the second cell, check `DATA_PATH`, `OUTPUT_DIR`, and `RUN_ORIGINS`, then run it (forecasts and evaluation for A, B, C, 2026_34; 32 sequential validation rounds per origin).
3. "Selected models and ensemble weights" cell -> confirm the top-2 models per origin and that the weights sum to 1.
4. "WIS summary" cell -> `ABC_WIS_summary.csv`, `ABC_WIS_by_week.csv`.
5. Per-origin plot cell -> `forecast_{A,B,C,2026_34}_edited.png`.
6. WIS comparison bar-chart cell -> `WIS_comparison.png`, `WIS_summary_all.csv`.
7. Last cell -> `submission_1B_long.csv` (origin 2026-34, task 1B).

#### 4-2. 1-c (`Problem1_1c.ipynb`)

1. Run the first code cell (function and constant definitions; includes `TREE_MIN_LEAF`, `TREE_MAX_FEATURES`, `BEST_LEAF`, `BEST_FEATURES` at the end of the cell).
2. In the second cell, edit `DATA_PATH` and `TEAM_ID`, then run it.
   * `RUN_SEARCH = True`: compares 3 settings, (leaf, max_features) = (3, 0.8), (2, 1.0), (1, 1.0), and copies the setting with the lowest ensemble validation WIS to `OUTPUT_DIR`.
   * `RUN_SEARCH = False`: runs only `BEST_LEAF=1`, `BEST_FEATURES=1.0`.
3. Third cell -> check candidate ranking and the forecast quantile table.
4. Plot cell -> `forecast_1C_edited.png` (the gray rolling-prediction line is cached in `training_predictions_*.csv`).

Before submitting: replace `TEAM_ID` (1-c notebook) and the `team_id` column in the long-format CSVs (`REPLACE_TEAM_ID`) with the team ID (e.g., `T07`).

### 5. Runtime

| Notebook | Runtime | Notes |
|---|---|---|
| 1-a / 1-b | **TODO: measure** | 4 origins x 32 sequential validation rounds x 5 candidates (160 trees). The notebook only says "a few minutes" |
| 1-c (`RUN_SEARCH=True`) | ~5 min | Per-setting 87.2 / 102.6 / 112.0 s from the run output (~302 s total). The notebook's "about 2 min 15 s" refers to the provided environment and differs from this one |
| 1-c (`RUN_SEARCH=False`) | ~1-2 min | One run of the best setting (measured 112 s) |

Runtime depends on CPU, so report the measured environment (CPU/RAM) alongside these figures.

### 6. Outputs

| File | Contents |
|---|---|
| `predictions.csv` | Seven quantiles per origin and target week (plus actuals where available) |
| `validation_comparison.csv` | Per-candidate validation WIS/RMSE, rank, selection flag, weight |
| `ensemble_weights.csv` | Selected top-2 models and their weights |
| `forecast_*_edited.png` | Observed / Trained (gray) / Observed test / Predicted / 95% and 50% PI |
