# Copilot instructions for this repo

Purpose: Help AI coding agents work productively on this short‑term PV generation forecasting project.

Big picture
- Single-notebook workflow in `PV_generation_forecasting.ipynb` for data prep, modeling, and evaluation.
- Data source: `data/time_series_15min_singleindex_filtered.csv` with 15‑min cadence from 2014‑12‑31 to 2020‑09‑30.
- Columns: `utc_timestamp` (ISO Z), `cet_cest_timestamp` (local with DST), `AT_solar_generation_actual` (target; floats, many night-time NaNs/zeros).
- Goal: build/compare short‑term regression models and visualize results.

Conventions and patterns
- Treat `utc_timestamp` as the canonical index; use timezone-aware parsing and resample to 15min if needed.
- Do not forward-fill overnight gaps; keep night values as NaN or zero per metric choice. Mask non-daylight for model training/eval where appropriate.
- Train/test split is time-based (no shuffle). Typical split: last N months (e.g., 2019‑2020) for test.
- Feature engineering: lags (e.g., 1–96 steps), rolling stats (mean/max), time-of-day/day-of-year cyclic encodings; avoid leakage by using only past data.
- Metrics: MAE, RMSE, R² on train/test; plots of predicted vs. actual time series.

Developer workflows
- Use the notebook. If adding scripts, keep them minimal and callable from cells.
- Recommended Python libs: pandas, numpy, scikit-learn, statsmodels, matplotlib/plotly. Pin versions if adding a requirements file.
- For quick peeks at the CSV (201k rows), read with `parse_dates=['utc_timestamp'], index_col='utc_timestamp'` and `dtype={'AT_solar_generation_actual':'float32'}` to reduce memory.

Data handling gotchas
- DST: prefer `utc_timestamp`; if using `cet_cest_timestamp`, localize/convert explicitly to avoid duplicated/missing times.
- Missing values: early morning/evening are often NaN; define a consistent rule (e.g., treat NaN as 0 only during astronomical night, otherwise drop/impute).
- Non-stationarity: strong annual seasonality; consider per‑month models or include day-length/solar elevation features.

Examples and file anchors
- Notebook: `PV_generation_forecasting.ipynb` (main workflow; add sections for preprocessing, features, models, results).
- Data: `data/time_series_15min_singleindex_filtered.csv` (3 columns; ~201,606 lines).

Good first enhancements for agents
- Add a preprocessing cell: load CSV, set UTC index, basic cleaning, and a day/night mask.
- Add baseline models: persistence (t+1 = t), SMA with window=4/8/16, and a simple Ridge/ARIMA comparison with time‑split.
- Add evaluation helpers: function to compute MAE/RMSE/R² and to plot a selected date range.

Style
- Keep cells small and labeled; avoid long side‑effects. Prefer pure functions defined once and reused.
- When adding new artifacts, document assumptions inline in the notebook.
