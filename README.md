# Fundamental Machine Learning Long/Short Equity Strategy
### A machine learning research project that uses company fundamentals and price features to generate predictive buy/sell signals.

## Overview
This project tests whether lagged company fundamentals from 10-Q filings, combined with features built from historical prices, can generate a useful machine learning signal for three-month stock returns.

Rather than trying to forecast the whole market, the project treats this as a binary classification problem: will an individual stock go up or down over the next three months? Right now, the focus is on evaluating how good these classifications are, using standard model performance metrics.

Future work would extend these model outputs into a formal long/short portfolio backtest, where positive signals are used as long candidates and negative signals are used as short candidates. This would allow the strategy to be evaluated against the S&P 500 benchmark using portfolio returns, drawdowns, and risk-adjusted performance metrics.

## Research Question
Can machine learning models trained on lagged 10-Q fundamentals data and historical price features generate predictive buy/sell signals for three-month stock returns?

## Key Results
The best model so far is an XGBoost classifier. It predicts whether a stock's return over the next three months will be positive or negative based on lagged company fundamentals and historical price features.

<p align="center">
  <img src="results/xgb_roc_auc_curve.png" alt="XGBoost ROC AUC Curve" width="650">
</p>

<p align="center">
  <img src="results/xgb_confusion_matrix.png" alt="XGBoost Confusion Matrix" width="520">
</p>

- The model scored 0.62 on ROC-AUC, modestly better than randomly guessing.
- The confusion matrix shows it can pick out both positive and negative return cases, but it still gets a meaningful number wrong.
- These numbers only measure how good the classifier is at separating the two classes. They don't tell us how a long/short portfolio built on these predictions would actually perform
- Returns, drawdowns, and risk-adjusted metrics are still on the to-do list.

## Data
Raw and cleaned datasets are not included in this repository due to licensing, redistribution, and file-size considerations.

This project used four datasets:

1. Financial Filings (10-Q) <br>
   Source: [SEC](https://www.sec.gov/data-research/sec-markets-data/financial-statement-data-sets) <br>
   Description: Quarterly company fundamental data extracted from SEC 10-Q filings, including income statement, balance sheet, and cash flow statement fields used to construct model features.

2. Standard Industrial Classification Codes (SIC Codes) <br>
   Source: [SEC](https://www.sec.gov/search-filings/standard-industrial-classification-sic-code-list) <br>
   Description: Industry classification data used as a categorial feature in the modeling phase and may eventually be used as a filter for L/S stocks of the same industry.

3. Historical Stock Prices <br>
   Source: [Yahoo Finance](https://finance.yahoo.com/) using the [yFinance](https://pypi.org/project/yfinance/) package <br>
   Description: Historical price data used to create price-based features and the binary target variable indicating whether a stock rose or fell over the following three-month period.

4. Market Capitalization Data <br>
   Source: [Stock Analysis](https://stockanalysis.com/list/biggest-companies/) <br>
   Description: Company market capitalization data used to support filtering during data cleaning and feature engineering.

## Methodology

### SEC Data Filtering and Cleaning
- Reduced the raw SEC financial statement dataset (~80 million rows) down to what was needed for analysis by:
   1. Keeping only rows representing grand totals, excluding subtotal rows
   2. Filtering to the tags relevant to the analysis (e.g., "Revenue", "OperatingIncomeLoss")
- Filtered the dataset to contain only unique ticker-tag-date-qtrs column combinations to avoid the duplicative rows that arise from YoY comparisons in subsequent 10-Q statements.

### SEC Missing Value Handling
- Combined related fields that describe the same data but under different labels (e.g., "Revenues" and "RevenuesFrom...Customer...").
- Reconstructed missing values for derivable features where possible (e.g., using COGS, R&D, SG&A, and D&A columns to populate "TopLineRevenue" and "OperatingIncome").
- Used linear interpolation to fill in remaining gaps on a company-by-company basis because their high level fundamentals shouldn't vary much from quarter to quarter.
- Dropped features with >50% missingness.
- Dropped the remaining handful of rows that contained a missing value.
- Removed any stocks with fewer than 20 observations. The final dataset contained 1,011 unique tickers spanning 179 months.
- Pivoted SEC financial statement tags into feature columns indexed by ticker and date.

### Dataset Joining and Target Creation
- Collected monthly Yahoo Finance price data by ticker, and saved local price files.
- Joined SEC financials with Yahoo Finance data, SEC SIC codes, and Stock Analysis market cap data to create one unified dataset enabling feature engineering and target variable creation.
- Created a five-category target variable before realizing that it spreads the data too thin. Revised this to a two-category target, 1 for when a stock has gone up in the following three months, and 0 for when it's down.

### Exploratory Data Analysis
- Identified very poor linear correlation of features with five-category target variable. This eventually informed the decision to drop to a two-category target.

### Feature Engineering
- Consolidated similar industry categories (e.g., "Finance" with "Finance or...Crypto Assets") to decrease cardinality for modeling.
- Built temporal features to show how fundamentals change quarter to quarter, careful not to leak future information to avoid look-ahead bias:
   - Lagged versions of fundamental variables
   - Three-period moving averages
   - Period-over-period changes in fundamentals and ratios
- Tested ticker encoding approaches to represent company-specific effects without relying only on raw ticker labels.
- Kept the financial ratio features. While they  didn't add much on their own, they normalize for company size and made business sense to include.


### Model Selection
- Compared Logistic Regression, Decision Tree, Random Forest, and XGBoost using a fixed time-based train/test split.
- Used F1 score to compare models because a long/short portfolio wants to correctly select from both the positive and negative classes.
- Selected XGBoost for the final stage because it performed competitively while running faster than Random Forest.
- Tested the final XGBoost model using an iterative walk-forward approach, where the model predicted the next period and was then retrained with newly available data.
- Looked at confusion matrices, precision, and recall alongside F1, since the type of error matters for stock selection, not just the overall score.

## Repository Structure
```text
.
├── data/
│   ├── README.md                         # Data download instructions and expected folder layout
│   ├── raw/                              # Raw SEC, price, and market cap files (not committed)
│   └── clean/                            # Cleaned modeling datasets (not committed)
├── results/                              # Model result figures used in the README
├── src/
│   ├── 01_sec_fundamentals_master_creation.ipynb
│   └── 02_full_project_report.ipynb
├── README.md
└── requirements.txt
```

## Reproducing the Project

The pipeline runs through two notebooks in `src/`:

1. `01_sec_fundamentals_master_creation.ipynb` builds the cleaned SEC fundamentals file from the SEC quarterly ZIP files you download manually.
2. `02_full_project_report.ipynb` combines those fundamentals with Yahoo Finance price data, builds the final modeling files, and runs the analysis, modeling, and results.

Run both notebooks from inside the `src/` folder so the relative paths work correctly. Raw and cleaned data aren't committed to the repo. (See `data/README.md` for the folder layout, manual downloads, and files each notebook generates).

## Technologies Used

- **Python**: Main language for cleaning data, building features, and running the models
- **Jupyter Notebook**: Used for analysis and reporting throughout the project
- **pandas / NumPy**: Clean, reshape, and compute over the data
- **scikit-learn**: Handles train/test splits, preprocessing, and model scoring
- **XGBoost**: Gradient boosting model used for the final predictions
- **yFinance**: Pulls historical stock price data
- **Matplotlib / Seaborn**: Used to generate charts and visualize model results

## Limitations and Future Work
- This project only evaluates classification performance. A full long/short portfolio backtest is still needed to measure actual returns, drawdowns, and risk-adjusted performance.
- Several financial statement fields that could have helped (e.g., CapEx, and FCF) were too sparse to use, which limited the final feature set.
- The final model beats random guessing, but it still produces a meaningful number of false positives and false negatives. There are numerous ways to try and rectify this, one being to add more features; if it's possible to find the data, adding the omitted features from above could be valuable. Another would be to conitue with model refinement and hyperparameter tuning.
- The original five-class target was too granular for the available data and produced weak results, so the project was narrowed down to binary buy/sell classification. An interesting area for deeper exploration would be to continue refining the feature set and model(s) and testing performance on a three, four, or even five class target.