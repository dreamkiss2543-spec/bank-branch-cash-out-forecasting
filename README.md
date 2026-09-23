# Bank Branch Cash-Out Forecasting

Machine-learning project for **one-day-ahead cash-out forecasting at bank branches**.

The project uses historical daily branch-level transaction data from **January 2018 to December 2020**.
After data-quality handling and time-series feature construction, the modeling dataset contains
**10,532 branch-day observations from 10 bank branches**.

Because the original data are confidential, **raw banking data and transaction-level outputs are not published**.

## Problem

Bank branches need enough cash to support customer withdrawals, while holding too much cash increases
storage, insurance, security, and transportation costs. The project therefore forecasts the next day's
total cash-out amount to support daily branch cash planning.

The target is:

```text
TOTAL_CASH_OUT = CASH_OUT_AMT + CASH_OUT_AMT_GT1M
```

The prediction setting is:

```text
information available at day t  ->  cash-out at day t+1
```

## Project Workflow

```text
Raw branch-day data
        |
        v
Data validation and cleaning
        |
        v
Missing branch-date audit
        |
        v
Daily calendar grid
        |
        v
Next-day target (t -> t+1)
        |
        v
Calendar + lag + rolling features
        |
        v
Chronological split
Subtrain -> Validation -> Final Test
        |
        v
Baseline and ML comparison
        |
        v
Residual modeling
Rolling7 + HistGradientBoosting
        |
        v
Final Test evaluation
        |
        v
Safety-buffer simulation
```

## Features

The notebook creates features from information available by the end of day `t`, including:

- Current-day cash variables
- Lag features: 1, 2, 3, 7, 14, and 28 days
- Rolling features: 3, 7, 14, and 30 days
- Day of week and month
- Weekend indicators
- Thai holiday indicators
- Pre-holiday and post-holiday indicators
- Payday / month-start / month-end indicators
- Branch identifiers

The workflow explicitly checks the feature set for missing values, infinite values, duplicate feature names,
and target leakage before model training.

## Data Split

The dataset is split in chronological order to preserve the time-series structure:

- **Subtrain:** first 64% of forecast dates
- **Validation:** next 16%
- **Final Test:** last 20%

The Final Test set is kept locked until model selection is complete.

## Models Compared

Historical baselines and machine-learning models include:

- Naive current-day cash-out
- Rolling mean (7 days)
- Rolling mean (30 days)
- Ridge Regression
- Random Forest
- Extra Trees
- HistGradientBoosting
- XGBoost
- Residual HistGradientBoosting models

## Final Method

The selected approach uses a **Rolling7 baseline** and predicts the remaining error with
**HistGradientBoosting**.

```text
Final Prediction
    =
Rolling7 Baseline
    +
Predicted Residual
```

Residual predictions are clipped using the **1st and 99th percentiles** estimated from the development data.

## Results

| Metric | Result |
|---|---:|
| Final model WAPE | **54.67%** |
| Rolling7 baseline WAPE | **71.15%** |
| WAPE reduction | **16.48 percentage points** |
| MAE | **1,354,264.22 THB** |
| RMSE | **2,847,928.10 THB** |

The final method was selected using the Validation set before the Final Test set was evaluated.

## Safety-Buffer Simulation

The forecast was also evaluated as a cash-planning input.

| Cash buffer | Under-coverage rate | Over-coverage rate |
|---:|---:|---:|
| 0% | 39.44% | 52.99% |
| 30% | 27.76% | 64.67% |

Increasing the safety buffer reduces under-coverage but increases over-coverage. The forecast is therefore
intended as **decision support**, not as an automatic cash-ordering rule.

## Repository Structure

```text
bank-branch-cash-out-forecasting/
│
├── README.md
├── README_TH.md
├── requirements.txt
├── .gitignore
├── bank_branch_cash_out_forecasting.ipynb
├── data_dictionary.csv
└── IS_Abstract.pdf
```

## How to Run

The notebook was developed for **Google Colab**.

1. Open `bank_branch_cash_out_forecasting.ipynb` in Google Colab.
2. Run the installation/import cell.
3. Upload the source CSV or Excel file when prompted.
4. Run the remaining cells in order.

The original banking dataset is required to reproduce the exact reported results.

## Technologies

- Python
- pandas
- NumPy
- scikit-learn
- XGBoost
- Matplotlib
- holidays
- joblib
- Google Colab

## Confidentiality

This public version intentionally excludes:

- Original raw banking data
- Real transaction values in notebook outputs
- Generated row-level prediction files
- Exported trained-model artifacts based on confidential data

Only the methodology, code, aggregate results, and project abstract are included.
