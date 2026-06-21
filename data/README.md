# Data Reproducibility

The raw and cleaned datasets aren't included in this repo because they're too big to commit, and some come with licensing or redistribution restrictions. To reproduce the project, build the folder structure below and run the notebooks in `src/`.

## Folder Layout

```text
data/
  raw/
    10Q_folder/
      zips/
        2010q1.zip
        ...
        2024q4.zip
      subfolders/
        2010q1/
        ...
        2024q4/
    price_batches/
  clean/
    all_stock_tickers.xlsx
    sic_codes.xlsx
```

## Files You Need to Get Manually

**1. SEC financial statement ZIP files**
- Get them from: https://www.sec.gov/data-research/sec-markets-data/financial-statement-data-sets
- Download every quarterly ZIP from `2010q1.zip` through `2024q4.zip` (or up to whatever the latest quarter is when you're running this).
- Put them in `data/raw/10Q_folder/zips/`.

**2. Ticker and market-cap data**
- Get it from: https://stockanalysis.com/stocks/
- Create a free account and download the full dataset.
- Save it as `data/clean/all_stock_tickers.xlsx`.
- It needs at least a `Symbol` column and a `Market Cap` column for the notebook to run.

**3. SIC code lookup**
- Get it from: https://www.sec.gov/search-filings/standard-industrial-classification-sic-code-list
- Copy and paste the table into a spreadsheet and save it as `data/clean/sic_codes.xlsx`.
- The report notebook joins this in using the `sic` column.

## Running the Notebooks

**Step 1 — `src/01_sec_fundamentals_master_creation.ipynb`**

This notebook:
- Unzips the SEC files from `data/raw/10Q_folder/zips/` into `data/raw/10Q_folder/subfolders/`.
- Reads `sub.txt` and `num.txt` from each quarter.
- Builds intermediate parquet files in `data/raw/10Q_folder/`.
- Saves the main output file you'll need for step 2:
  ```text
  data/clean/num_sub_master_recon.parquet
  ```

**Step 2 — `src/02_full_project_report.ipynb`**

This notebook:
- Reads `data/clean/num_sub_master_recon.parquet` and `data/clean/all_stock_tickers.xlsx`.
- Pulls monthly price data using `yfinance`.
- Saves individual price files to `data/raw/price_batches/`.
- Combines them into `data/clean/master_price_data.csv`.
- Builds `data/clean/master_file.csv` and `data/clean/master_file_v2.csv`.
- Reads in `data/clean/sic_codes.xlsx` and runs the final analysis and modeling.

## A Few Things to Know

- Run the notebooks from inside the `src/` folder. The relative paths (like `../data/clean/...`) assume that's your working directory.
- Price data comes from Yahoo Finance via `yfinance`. If Yahoo updates its historical data or rate-limits you, your numbers might come out slightly different.
- Building the SEC `num_master` file takes a while since there's a lot of data. That's also why it's saved as parquet instead of CSV.