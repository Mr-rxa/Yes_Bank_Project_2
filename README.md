# Yes Bank Stock Price Analysis and Machine Learning Project

An end-to-end, educational data science project on **185 monthly observations of Yes Bank stock prices from July 2005 to November 2020**. It combines exploratory data analysis, statistical hypothesis testing, feature engineering, regression modelling, explainability, a saved estimator, and a separate rolling-origin evaluation of genuine next-month forecasts.

> **Important distinction:** the saved Linear Regression model estimates a month's close using that same month's High and Low. It cannot make a forecast before those values are known. The separate next-month forecasting experiment uses lagged information only and does not outperform a simple persistence benchmark overall.

## Project information

| Field | Details |
|---|---|
| Student | Rahul Sharma |
| Programme | B.Tech Information Technology (Hons.) |
| Institution | Panipat Institute of Engineering and Technology (PIET), Samalkha |
| GitHub profile | https://github.com/Mr-rxa |
| Contribution | Individual |
| Dataset period | July 2005-November 2020 |
| Dataset size | 185 monthly rows; 5 columns |

A project-specific repository URL was not present in the submitted materials. The link above is the author's GitHub profile, not a claim that a repository has already been published.

## Repository contents

```text
Yes_Bank_Project/
├── README.md
├── requirements.txt
├── MODEL_CARD.md
├── Yes_Bank_EDA_Submission.ipynb
├── Yes_Bank_ML_Submission.ipynb
├── Yes_Bank_Report.pdf
├── Yes_Bank_Project_Presentation.pptx
├── data_YesBank_StockPrices.csv
├── figures/
│   ├── yes_bank_close_history.png
│   └── rolling_origin_conformal_intervals.png
└── models/
    └── yes_bank_close_model.joblib
```

## Dataset

The raw file **`data_YesBank_StockPrices.csv` is included in this ZIP**. It contains `Date`, `Open`, `High`, `Low` and `Close` columns, with 185 monthly observations. The source archive did not document an external download URL, so this README does not invent one. Keep the CSV in the project root beside the notebooks.

## Notebooks and deliverables

### 1. `Yes_Bank_EDA_Submission.ipynb`

- Data inspection, validation and wrangling.
- 20 visualizations covering price trends, distributions, returns, volatility, drawdowns, OHLC relationships and engineered indicators.
- Chart-level observations, implications, recommendations and an EDA conclusion.

### 2. `Yes_Bank_ML_Submission.ipynb`

- 15 core ML visualizations, feature engineering and 3 hypothesis tests.
- Chronological train/test split for the in-month close-estimation task.
- Linear Regression, Ridge, Random Forest and Gradient Boosting comparisons, with time-series cross-validation and tuning.
- Explainability with coefficients and permutation/feature importance.
- A separate **three-fold rolling-origin next-month forecast evaluation**, feature ablation and 90% split-conformal prediction bands.

### 3. `Yes_Bank_Report.pdf`

A concise two-page project report summarizing the dataset, EDA, in-month model results, rolling-origin forecast evaluation, conformal interval coverage and limitations.

### 4. `Yes_Bank_Project_Presentation.pptx`

A short presentation deck covering the problem, dataset, EDA story, in-month estimator, rolling-origin forecasting results and limitations.

### 5. `MODEL_CARD.md`

Documents the saved model's intended use, target, inputs, training/evaluation periods, performance, limitations and the separate next-month forecast diagnostic.

## Main findings

- The series peaks at a monthly close of INR 367.90 in July 2018 and ends at INR 14.67 in November 2020, approximately 96% below the peak.
- The raw data contains no missing values or duplicate monthly rows; all 185 months from July 2005 through November 2020 are present.
- The rise and decline periods have different return and volatility behaviour, so full-period averages should be interpreted carefully.
- The in-month model uses two scale-free features: `log(High / Open)` and `log(Low / Open)`. The saved model is a Linear Regression estimator for `log(Close / Open)`.

## In-month close-estimation results

The model is trained on the first 148 months (July 2005-October 2017) and evaluated on the final 37 months (November 2017-November 2020), preserving chronology.

| Metric | Linear Regression | Open = Close benchmark |
|---|---:|---:|
| R-squared | 0.9867 | 0.9085 |
| RMSE | INR 14.79 | INR 38.76 |
| MAE | INR 9.43 | Not reported |
| MAPE | 8.35% | 22.13% |

These scores apply **only after the same month's High and Low are known**. They are not next-month forecasting scores.

## Genuine next-month forecasting: rolling-origin evaluation

To avoid relying on a single three-year holdout, the ML notebook now includes three expanding rolling-origin folds. Each fold fits the model on prior observations, calibrates an interval on the next 16 months, then tests on the following 10 months. Across folds, 30 distinct one-month-ahead forecasts are evaluated. Features are selected using expanding-window cross-validation within the first training prefix only.

**Feature ablation (mean CV RMSE, INR):**

| Feature set | Mean CV RMSE |
|---|---:|
| Return lag 1 | **7.74** |
| Return lag 1 + 3-month momentum | 7.82 |
| Return lag 1 + 3- and 6-month momentum | 8.26 |

The parsimonious one-feature Ridge model is then compared with persistence (next close equals previous close):

| Rolling test window | Ridge RMSE | Persistence RMSE | Ridge MAPE | Persistence MAPE |
|---|---:|---:|---:|---:|
| Oct 2015-Jul 2016 | INR 14.30 | INR 16.65 | 6.00% | 7.48% |
| Dec 2017-Sep 2018 | INR 59.89 | INR 57.88 | 16.42% | 15.37% |
| Feb-Nov 2020 | INR 6.50 | INR 6.33 | 28.25% | 25.76% |
| **Pooled (30 months)** | **INR 35.75** | **INR 34.97** | **16.89%** | **16.21%** |

See `figures/rolling_origin_conformal_intervals.png` for the forecast and band visualization.

**Conformal intervals:** the fold-specific 90% bands covered 27 of 30 holdout months (90% empirical coverage). Their average width was INR 76.22, about 75.3% of the actual close on average. These wide bands indicate substantial uncertainty. Because the calibration samples are small and financial time series are not exchangeable across regime changes, this observed coverage is not a guarantee of future coverage.

The lagged-feature Ridge model does **not** beat persistence on the pooled metrics. The evidence does not support claiming a reliable next-month forecast from this dataset.

## Installation and execution

Python 3.11 or later is recommended.

```bash
# 1. Extract the ZIP and open a terminal in the Yes_Bank_Project folder

# 2. Create and activate a virtual environment (recommended)
python -m venv .venv

# Windows PowerShell
.venv\Scripts\Activate.ps1

# macOS / Linux
source .venv/bin/activate

# 3. Install the tested dependencies
python -m pip install --upgrade pip
pip install -r requirements.txt

# 4. Start Jupyter
jupyter notebook
```

Open either notebook from the project root and run the cells from top to bottom. The CSV is loaded using a relative path from the project root; no `/mnt/data` paths are required. The ML notebook writes the saved model under `models/` and the forecast chart under `figures/` when the relevant cells are run.

## Load the saved estimator

```python
import joblib
import numpy as np
import pandas as pd

artifact = joblib.load("models/yes_bank_close_model.joblib")

def estimate_monthly_close(open_price: float, high: float, low: float) -> float:
    if not (0 < low <= open_price <= high):
        raise ValueError("Expected 0 < Low <= Open <= High, in INR.")

    X = pd.DataFrame({
        "High_Rel": [np.log(high / open_price)],
        "Low_Rel": [np.log(low / open_price)],
    })
    log_close_open = artifact["model"].predict(X[artifact["features"]])[0]
    return float(open_price * np.exp(log_close_open))

# Example only: same-month High and Low must already be known.
print(estimate_monthly_close(open_price=14.0, high=15.5, low=12.5))
```

Use the scikit-learn version pinned in `requirements.txt` when loading the serialized model. Only load Joblib artifacts from trusted sources.

## Limitations and responsible use

- The dataset ends in November 2020 and is not current market data.
- Only 185 monthly observations are available; conclusions and validation results are sample-limited.
- The saved model uses same-month High and Low and is therefore not a pre-month forecast.
- The true next-month experiment does not beat the persistence benchmark overall.
- Conformal intervals are wide and calibrated from small chronological windows; their historical coverage should not be treated as a future guarantee.
- The dataset omits broader market indices, fundamentals, macroeconomic conditions, corporate actions and news variables.

This repository is for **educational and analytical use only**. It is not investment advice and should not be used on its own to make trading or investment decisions.
