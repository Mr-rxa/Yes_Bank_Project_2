# Model Card - Yes Bank Monthly Close Estimator

## Model identity

- **Artifact:** `models/yes_bank_close_model.joblib`
- **Algorithm:** scikit-learn `LinearRegression`
- **Task:** Estimate the current month's closing price from the same month's Open, High and Low prices
- **Target:** `log(Close / Open)`
- **Selected input features:** `High_Rel = log(High / Open)` and `Low_Rel = log(Low / Open)`
- **Training period:** Jul-2005 to Oct-2017 (148 months)
- **Held-out period:** Nov-2017 to Nov-2020 (37 months)
- **Serialization environment:** scikit-learn 1.8.0; see `requirements.txt`

## Intended use

This artifact is for educational analysis and an **in-month estimate** after the month's High and Low values are available. It may be used to demonstrate regression, feature engineering, model persistence and interpretation on historical data.

## Out-of-scope use

This artifact is **not** a next-month forecasting model. High and Low for the target month are not known at the start of that month. Do not describe the 8.35% test MAPE as future forecasting accuracy. Do not use the artifact as a standalone basis for investing, trading, credit decisions or financial advice.

## Data

The bundled file `data_YesBank_StockPrices.csv` contains 185 monthly observations from July 2005 through November 2020 with Date, Open, High, Low and Close columns. The model is trained on the first 148 months and evaluated on the final 37 months in chronological order. No random shuffle is used.

## Performance on the chronological holdout

| Metric | Linear Regression | Open = Close benchmark |
|---|---:|---:|
| R-squared | 0.9867 | 0.9085 (reported) |
| RMSE | INR 14.79 | INR 38.76 (reported) |
| MAE | INR 9.43 | Not reported |
| MAPE | 8.35% | 22.13% (reported) |

These metrics describe estimation after same-month High and Low are observed. They do not quantify next-month forecasting ability.

## Separate next-month forecasting diagnostic

A second experiment uses lagged information only and evaluates three expanding rolling-origin folds. Feature ablation on the first training prefix compares one lagged return with additional 3- and 6-month momentum terms. The one-feature set (`Return_Lag_1`) has the lowest mean expanding-window CV RMSE (INR 7.74), versus INR 7.82 with 3-month momentum and INR 8.26 with both 3- and 6-month momentum. A Ridge model with that feature is then evaluated against persistence.

| Fold | Train months | Calibration period | Test period | Ridge RMSE | Persistence RMSE | Ridge MAPE | Persistence MAPE | 90% coverage |
|---:|---:|---|---|---:|---:|---:|---:|---:|
| 1 | 100 | Jun 2014-Sep 2015 | Oct 2015-Jul 2016 | INR 14.30 | INR 16.65 | 6.00% | 7.48% | 90% |
| 2 | 126 | Aug 2016-Nov 2017 | Dec 2017-Sep 2018 | INR 59.89 | INR 57.88 | 16.42% | 15.37% | 90% |
| 3 | 152 | Oct 2018-Jan 2020 | Feb-Nov 2020 | INR 6.50 | INR 6.33 | 28.25% | 25.76% | 90% |
| **Pooled (30 months)** | - | - | - | **INR 35.75** | **INR 34.97** | **16.89%** | **16.21%** | **90% (27/30)** |

The Ridge forecast does not beat persistence on pooled RMSE or MAPE. The 90% split-conformal bands have an average width of INR 76.22 (about 75.3% of actual close on average). The bands are therefore broad. Calibration uses 16 months per fold and the test uses 10; the time-series setting is not exchangeable, so nominal conformal coverage is not guaranteed in future regimes.

## Limitations and risks

- Only 185 monthly observations are available, ending in November 2020; this is not current market data.
- The market experienced a major regime change around 2018, and model performance varies substantially by period.
- The saved in-month estimator uses information that is unavailable before the month has unfolded.
- The next-month diagnostic does not outperform the persistence benchmark overall.
- The conformal calibration samples are small, and serial dependence/regime shifts weaken standard coverage assumptions.
- The data do not include broader market indices, fundamentals, macroeconomic variables, corporate actions or news.
- Joblib files should only be loaded from trusted sources.

## Monitoring and update policy

Do not deploy this model for financial decisions. Any future research version should use a more recent, longer dataset; nested chronological model selection; repeated rolling-origin tests; explicit baselines; and interval calibration rechecked after every regime change. Refit and re-evaluate before reuse rather than carrying these historical scores forward.
