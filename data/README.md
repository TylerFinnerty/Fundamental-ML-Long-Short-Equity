# Data Reproducibility

Raw and cleaned datasets are not committed to this repository because of licensing, redistribution, and file-size constraints. To reproduce the project, recreate the folder layout below and run the notebooks from the `src/` folder.

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

## Required Manual Inputs

1. SEC financial statement ZIP files
   - Source: https://www.sec.gov/data-research/sec-markets-data/financial-statement-data-sets
   - Download every quarterly ZIP file from `2010q1.zip` through `2024q4.zip` (or your desired end date if performing this analysis far in the future).
   - Place the ZIP files in `data/raw/10Q_folder/zips/`.

2. Ticker and market-cap file
   - Source: https://stockanalysis.com/stocks/
   - Create a free account and download the data full data set
   - Save the ticker list as `data/clean/all_stock_tickers.xlsx`.
   - The notebook expects at least `Symbol` and `Market Cap` columns.

3. SIC code file
   - Source: https://www.sec.gov/search-filings/standard-industrial-classification-sic-code-list
   - Save the SIC code lookup as `data/clean/sic_codes.xlsx` by copying and pasting from the website.
   - The report notebook merges this file on `sic`.

## Notebook Steps

1. Run `src/01_sec_fundamentals_master_creation.ipynb`.
   - Extracts SEC ZIP files from `data/raw/10Q_folder/zips/` into `data/raw/10Q_folder/subfolders/`.
   - Reads each quarter's `sub.txt` and `num.txt`.
   - Creates SEC intermediate parquet files in `data/raw/10Q_folder/`.
   - Writes the main handoff file:
     ```text
     data/clean/num_sub_master_recon.parquet
     ```

2. Run `src/02_full_project_report.ipynb`.
   - Reads `data/clean/num_sub_master_recon.parquet`.
   - Reads `data/clean/all_stock_tickers.xlsx`.
   - Downloads monthly price data with `yfinance`.
   - Writes individual price parquet files to:
     ```text
     data/raw/price_batches/
     ```
   - Consolidates price data into:
     ```text
     data/clean/master_price_data.csv
     ```
   - Creates:
     ```text
     data/clean/master_file.csv
     data/clean/master_file_v2.csv
     ```
   - Reads `data/clean/sic_codes.xlsx` and runs the final analysis and modeling sections.

## Notes

- Run notebooks from the `src/` folder. Paths such as `../data/clean/...` assume that working directory.
- Yahoo Finance data is collected through `yfinance`; exact results may vary if Yahoo revises historical data or rate-limits requests.
- The SEC `num_master` build is large and can take a long time. Parquet is used instead of CSV for large intermediate files.
