# SHORT-TERM_FORECASTING_OF_PHOTOVOLTAIC_POWER_GENERATION

## 1. RELEVANCE

Short‑term PV generation forecasts (minutes to days ahead) help grid operators and market participants balance supply and demand, schedule reserves, reduce curtailment, and optimize storage/dispatch. Accurate intraday predictions at 15‑minute cadence reduce imbalance costs and enable better utilization of renewable assets.


## 2. INITIAL DATA AND PREPROCESSING

Data: Open Power System Data (OPSD) time series for Austria (AT), filtered to 15‑minute cadence.

- File: `data/time_series_15min_singleindex_filtered.csv`
- Columns used in this project:
	- `utc_timestamp` (ISO UTC, canonical index)
	- `AT_solar_generation_actual` (target; MW)
- Date range: 2014‑12‑31 to 2020‑09‑30 (approx., see notebook for exact min/max)

Loading and basic cleaning are implemented in `PV_generation_forecasting.ipynb`:

- Parse `utc_timestamp` and set it as the DataFrame index.
- Use `float32` for the target to reduce memory footprint.
- Policy: standardize night‑time NaNs to 0 (this project treats all NaNs as 0 initially; can refine with a solar‑elevation mask).
- Optional: clip small negative values to 0.

Caveats and conventions:

- Prefer UTC index to avoid DST ambiguity. If using local time, localize/convert explicitly.
- Do not forward‑fill across nights. Keep zeros overnight; model training can use masks if needed.
- Resampling to 15‑min is not forced unless necessary; the file already has 15‑min cadence.

## 3. DATA VISUALIZATION (1/2)

Typical plots (examples are in the notebook):

- Daily/weekly time windows of the target to inspect variability and noise.
- Yearly overview (downsampled) to see strong annual seasonality and winter/summer asymmetry.
- Histogram or KDE of daytime values vs. full‑day values to see the effect of zeros at night.

## 4. DATA VISUALIZATION (2/2)

Feature diagnostics:

- Lag plots: `y(t)` vs `y(t-1)`, `y(t-4)`, `y(t-96)`.
- Rolling statistics: 1h/2h/24h moving means as simple smoothers.
- Cyclic encodings: hour‑of‑day and day‑of‑year sin/cos features.

## 5. REGRESSION MODELS

Implemented in the notebook with a 7/2/1 chronological split (train/validation/test):

- Baseline: Persistence (predict `t+1 = t` via `lag_1`).
- Baseline: Simple Moving Average (SMA) over last 4 steps (1 hour).
- Linear model: Ridge regression on engineered features (lags 1/4/8/96, rolling means, cyclic time features).
- Time‑series model: SARIMAX with daily seasonality (`m=96`) as an ARIMA‑style baseline.

Optional next models:

- RandomForest/XGBoost regressors on the same feature set.
- Exogenous SARIMAX leveraging engineered features.
 
## 6. FORECASTING RESULTS

Evaluation metrics used:

- MAE, RMSE, R² on validation and test splits.

Visualizations:

- Overlay of predictions vs. actuals for a recent 7‑day test window (lines for Actual, Persistence/SMA, Ridge, SARIMAX).
- Additional windows or seasonal slices can be added for diagnostics.

## 7. CONCLUSION

This repository provides a compact baseline for short‑term PV forecasting at 15‑minute cadence. It standardizes night‑time NaNs to 0, uses a chronological 7/2/1 split, and compares simple yet strong baselines (Persistence, SMA, Ridge) with a seasonal ARIMA (SARIMAX) model. The notebook is structured to make it easy to add features, swap models, and iterate on evaluation. Future improvements can incorporate weather forecasts, solar elevation masks, and model ensembling.

---

### Repo structure

- `PV_generation_forecasting.ipynb` — Main workflow (preprocessing, features, models, evaluation).
- `data/time_series_15min_singleindex_filtered.csv` — Input time series data (15‑min cadence).

