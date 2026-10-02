## Problem 2: 2026-27 Season Influenza and COVID-19 Long-Range Forecasts

### 1. Overview

| Item | Description |
|---|---|
| Targets | Influenza: `ili` (weekly ILI per 1,000 outpatients, 300-site basis) and `flu_hosp` (influenza hospitalizations, `ARI` column). COVID-19: `covid_ari` (hospitalizations, `ARI` column) and `covid_sari` (optional, `SARI` column). National, all ages, weekly |
| Origin / horizon | Origin 2026-39; 49 weeks, 2026-40 to 2027-35 |
| Quantiles | 0.025, 0.10, 0.25, 0.50, 0.75, 0.90, 0.975 |
| Common scenarios | Influenza **F1** (no new subtype; A(H3N2)+B detection < 5% all season) and **F2** (A(H3N2) onset around 2026-48 +/- 2 weeks, 2025-26 season growth). COVID-19 **C1** (no new variant becomes dominant; PQ.16.1.1 persists) and **C2** (a new variant reaches 50% share around 2026-50 +/- 2 weeks). 1,000 paths each |
| Representative forecast (REP) | Equal-weight mixture of the two scenarios of each pathogen: the 1,000 + 1,000 scenario paths are pooled (2,000 paths), so each scenario has weight exactly 1/2. The branching conditions (A(H3N2) detection after 2026-48, new-variant dominance) cannot be observed at the origin, so there is no basis for favoring either scenario and a more precise weight would have no support |
| Model | Level model + AR(1) error. Influenza: `log1p(target)` explained by same-week detection rates (overall, A(H1N1)pdm09, A(H3N2), B) and a seasonal sin/cos. COVID-19: `log1p(target)` explained by four variant-share features (dominant lineage share, new-lineage share, 4- and 12-week turnover) and two seasonal harmonics. Candidates are Ridge, ExtraTrees (ET) and (influenza only) HistGradientBoosting, averaged in log space |
| Selection | Rolling backtest with the *actual* detection rates / variant shares as input. Influenza: Ridge+ET, innovation scale 1.25, relative WIS 0.289. COVID-19: Ridge+ET, long-run error sd 0.8, relative WIS 0.845 (both against the Appendix B baseline). The hospitalization (`flu_hosp`) and SARI targets reuse the selection made for ILI and ARI (no separate backtest) |
| Prediction intervals | Each path = level forecast + decaying last error + AR(1) random error. Quantiles are taken over the 1,000 (scenario) or 2,000 (REP) paths |
| Information cutoff | Training, validation and selection use data up to the origin 2026-39 only. Missing values are not imputed (the notebooks stop if required columns have missing values). COVID-19 variant shares end in 2026-08 and the last share is held afterwards |
| Policy briefing | Q-a (simultaneous peak), Q-b (decision rule for weeks 2026-40 to 43), Q-c (next summer) are computed from the REP paths. The influenza and COVID-19 paths are paired at random (independent) |
| Reproducibility | `SEED = 2026`, `n_jobs=1`, `OMP/OPENBLAS_NUM_THREADS=1` |

### 2. Notebooks, Folder / Files

Three notebooks make the Problem 2 answers. **The notebooks in this folder are copies of the working notebooks. They read and write files by relative paths, so run them from the original locations (or recreate the layout below).**

| Notebook (this folder) | Original (working) notebook | Role | Writes |
|---|---|---|---|
| `Problem2_influenza_scenarios.ipynb` | `influenza/Problem2_F1_F2_longrange_v3_w50.ipynb` | Influenza F1, F2, REP for `ili` and `flu_hosp` | `influenza/results/` |
| `Problem2_covid_scenarios.ipynb` | `covid/Problem2_C1_C2_covid_v2.ipynb` | COVID-19 C1, C2, REP for `covid_ari` and `covid_sari` | `covid/results_v2/` |
| `Problem2_brief_and_submission.ipynb` | `Problem2_brief_and_submission_w50.ipynb` | Policy-briefing numbers and figures, fills the submission workbook | `submission/` |

Original location: `Code/Q2_ensemble/` (`D:\Dropbox\Jihyeon\MSOL_research\2609_Forecast_workshop\Code\Q2_ensemble`). Layout the notebooks expect (the names of the notebooks may be either of the two columns above):

```
Q2_ensemble/
  Problem2_brief_and_submission_w50.ipynb
  influenza/
    Problem2_F1_F2_longrange_v3_w50.ipynb
    인플루엔자 2차 학습 데이터_아형 검출률 추가.xlsx
    results/                         (created)
  covid/
    Problem2_C1_C2_covid_v2.ipynb
    코로나19 2차 학습 데이터.xlsx
    2020-2026.8 코로나19 변이바이러스 점유율 현황.xlsx
    results_v2/                      (created)
  submission/
    submissoin_T01.xlsx              (submission workbook; updated by the brief notebook)
```

Required data columns. Influenza: `년주, 주차, ILI, ARI, 전체 검출률(%), A(H1N1)pdm09, A(H3N2), B` (no missing values up to 2026-39). COVID-19: `년도, 주차, 년주, ARI, SARI` plus the monthly variant-share table. Place each Excel file in the **same folder** as its notebook. Scenario descriptions are in `시나리오설정_인플루엔자.docx` and `시나리오설정_코로나19_v3_REP반영.docx` (in the original folders).

### 3. Environment

* Python 3.13.11 (Jupyter kernel `python3`), CPU only; Windows 11
* Packages: `numpy 2.4.0, pandas 2.3.3, scikit-learn 1.8.0, scipy 1.16.3, matplotlib 3.10.8, openpyxl 3.1.5`; `jupyter`/`nbconvert` to run from the command line; `Pillow` is needed by `openpyxl` to embed the figure in sheet 8
* Font: `Malgun Gothic` (figure labels are Korean; without it the labels show as boxes but the numbers are unaffected)
* `requirements.txt`: just before submission, save the output of `pip freeze` for the packages above **(TODO: not yet recorded)**

### 4. Execution Order

Run the three notebooks **in this order**, each from its own folder (all cells top to bottom):

```bash
cd Code/Q2_ensemble/influenza
jupyter nbconvert --to notebook --execute --inplace Problem2_F1_F2_longrange_v3_w50.ipynb
cd ../covid
jupyter nbconvert --to notebook --execute --inplace Problem2_C1_C2_covid_v2.ipynb
cd ..
jupyter nbconvert --to notebook --execute --inplace Problem2_brief_and_submission_w50.ipynb
```

Close `submission/submissoin_T01.xlsx` in Excel before the last step (an open file cannot be saved; the notebook then stops with a message).

#### 4-1. Influenza (`Problem2_influenza_scenarios.ipynb`)

| Cell | What it does | Writes to `influenza/results/` |
|---|---|---|
| 1 | Imports, constants (`SEED`, `ORIGIN=202639`, `N_STEPS=49`, `N_PATHS=1000`, `W_F2=0.5`), data loading | |
| 3 | Level model, AR(1) error, simulation functions | |
| 5 | Rolling backtest (`BACKTEST_STEP=12`), model set and innovation scale selection, relative WIS | `backtest_by_step.csv`, `backtest_summary.csv`, `relative_WIS_by_horizon.csv`, `relative_WIS_by_origin.csv`, `relative_WIS_vs_baseline.png` |
| 7 | Scenario detection-rate paths for F1 and F2 | |
| 9 | Final training, 1,000 paths per scenario, REP (F1+F2 pooled), summaries | `timeseries_quantiles.csv`, `single_values.csv`, `paths.npz`, `detection_rate_inputs.csv`, `h3b_prior_year_inputs.csv`, `scenario_spec.json` |
| 11 | Figures | `scenario_F1.png`, `scenario_F2.png`, `scenario_comparison.png` |

Scenario inputs: A(H1N1)pdm09 follows past H1-dominant wave curves aligned to the current level (one drawn per path), in both scenarios. A(H3N2) and B: in F1, for each forecast week the same week one year earlier (then two years earlier) is used if A(H3N2)+B < 5%, otherwise a background level; in F2, B is as in F1 and A(H3N2) rises around 2026-48 (+/- 2 weeks) following the 2025-26 season's growth ratios.

#### 4-2. COVID-19 (`Problem2_covid_scenarios.ipynb`)

| Cell | What it does | Writes to `covid/results_v2/` |
|---|---|---|
| 1 | Imports, constants (`SEED`, `ORIGIN=202639`, `N_STEPS=49`, `N_PATHS=1000`, `TARGETS`, scenario parameters), data loading | |
| 3 | Share-based features, level model, AR(1) error (`col` selects ARI or SARI) | |
| 5 | Rolling backtest on ARI (14 origins from 2025-01, `BACKTEST_STEP=6`), model and `sinf` selection, relative WIS | `backtest_by_step.csv`, `backtest_summary.csv`, `relative_WIS_by_horizon.csv`, `relative_WIS_by_origin.csv`, `relative_WIS_vs_baseline.png` |
| 7 | Past dominant variants, C2 speed (5 replacement events), variant-share paths | `analog_patterns.csv`, `replacement_events.csv` |
| 9 | Final training (ARI and SARI separately), simulation for C1, C2, REP (C1+C2 pooled), summaries | `timeseries_quantiles.csv`, `single_values.csv`, `paths.npz`, `variant_share_inputs.csv`, `scenario_spec.json` |
| 11 | Figures | `scenario_C1.png`, `scenario_C2.png`, `scenario_REP.png`, `scenario_comparison.png` |

Variant-share paths are built from the 10 past dominant variants (Delta onward) in the monthly share table. C1 multiplies PQ.16.1.1's share (53.9% in 2026-08) by the share-ratio curve seen *before* a new variant arrived and holds the last ratio afterwards. C2 raises a new variant to 50% share in 2026-50 +/- 2 weeks at the median speed of 5 past replacements, then follows a past post-peak decline curve. Each scenario has 1,000 curves (small multiplicative noise, log sd 7%). The level model and AR(1) coefficient are fitted separately for ARI (phi 0.90, capped by `AR_PHI_MAX`) and SARI (phi 0.779); both share the same share paths.

#### 4-3. Policy briefing and workbook (`Problem2_brief_and_submission.ipynb`)

| Cell | What it does | Writes |
|---|---|---|
| 1 | Imports, paths, loads results, **checks that REP equals the pooled scenario paths** (stops otherwise: rerun the two notebooks above) | |
| 3 | Q-a: same-month peak probability, combined hospitalization peak | |
| 5 | Q-b: decision bands for ILI and COVID-19 hospitalization over 2026-40 to 43 | |
| 7 | Q-c: probability that ILI exceeds the epidemic threshold (12.9) in 2027-27 to 35 | |
| 9, 10 | Briefing figures | `submission/brief_figure.png`, `brief_figure_abc.png` |
| 12 | Fills sheets 3, 4, 7 and 8 of the workbook, saves it, checks the format (missing, negative, non-monotone quantiles) | `submission/submissoin_T01.xlsx`, backup in `submission/previous_run/` |
| 14 | Representative result (1/2 mixture) figure and tables | `submission/representative_equal_1x2.png`, `representative_equal_quantiles.csv`, `representative_equal_single_values.csv` |

Workbook handling: the base file is `submission/submissoin_T01.xlsx` if it exists, otherwise `submission/submissoin_T01_previous.xlsx`; only if neither exists does it start from the blank template (`../../자료/답안제출양식_2026추계워크숍.xlsx`) and it prints a warning because sheets 1 and 2 would then be empty. Sheets 0, 1 and 2 (guide, team information, Problem 1) are never modified. `covid_sari` rows (optional) are filled in sheets 3 and 4 for REP, C1 and C2. Before saving, the base workbook is copied to `submission/previous_run/`. The values of REP in sheets 3 and 4 equal `submission/representative_equal_*.csv`.

### 5. Runtime

| Notebook | Runtime | Notes |
|---|---|---|
| Influenza | ~2 min (111 s measured) | Full run including the backtest. Measured on a 16-logical-core PC while the COVID-19 notebook ran at the same time |
| COVID-19 | ~1 min (36-51 s measured) | Full run including the backtest |
| Policy briefing and workbook | ~20 s (18-19 s measured) | Reads the results and writes the workbook |

Runtime depends on CPU, so report the measured environment (CPU/RAM) alongside these figures **(TODO: record CPU model and RAM)**.

### 6. Outputs

| File | Contents |
|---|---|
| `influenza/results/timeseries_quantiles.csv` | Weekly seven quantiles: `target` (`ili`, `flu_hosp`) x `scenario_id` (`F1`, `F2`, `REP`) x 49 weeks |
| `covid/results_v2/timeseries_quantiles.csv` | Weekly seven quantiles: `target` (`covid_ari`, `covid_sari`) x `scenario_id` (`C1`, `C2`, `REP`) x 49 weeks |
| `*/single_values.csv` | Seven quantiles of `*_peak_week` (week index, 2026-40 = 1), `*_peak_value`, `*_cum_total` for each target and scenario |
| `*/paths.npz` | Simulated paths per target and scenario (REP has 2,000 paths) and the scenario detection-rate (`det_*`) or variant-share (`shares_*`) paths |
| `*/scenario_spec.json` | Settings, selected model, scenario details, REP weight and method, peak summaries |
| `influenza/results/detection_rate_inputs.csv`, `h3b_prior_year_inputs.csv` | Scenario detection-rate inputs and the source weeks used for A(H3N2)+B in F1 |
| `covid/results_v2/variant_share_inputs.csv`, `analog_patterns.csv`, `replacement_events.csv` | Scenario variant-share inputs, the 10 past dominant variants, the 5 replacement events |
| `*/backtest_*.csv`, `*/relative_WIS_*` | Backtest results and WIS relative to the Appendix B baseline |
| `*/scenario_*.png` | Scenario inputs and forecasts with 50% and 95% bands, and comparison figures |
| `submission/submissoin_T01.xlsx` | Submission workbook (sheets 3, 4, 7, 8 refreshed by the briefing notebook) |
| `submission/brief_figure.png`, `brief_figure_abc.png` | Policy-briefing figures |
| `submission/representative_equal_1x2.png`, `representative_equal_quantiles.csv`, `representative_equal_single_values.csv` | Representative forecast figure (ILI and COVID-19 ARI, 1/2 mixture, median with 50% and 95% bands) and its values |
