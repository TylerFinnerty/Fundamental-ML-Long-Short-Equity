# Fundamental Machine Learning Long/Short Equity Strategy
### A machine learning research project that uses company fundamentals and price features to generate predictive buy/sell signals.

## Overview
This project tests whether lagged company fundamentals from 10-Q filings combined with features derived from historical prices can be used to generate machine learning signals for three-month stock returns.

Rather than attempting to forecast the entire market, the project frames the problem as a binary classification task: predicting whether an individual stock is more likely to rise or fall over the next three months. The current version focuses on evaluating the quality of these buy/sell classifications using model performance metrics.

Future work would extend these model outputs into a formal long/short portfolio backtest, where positive signals are used as long candidates and negative signals are used as short candidates. This would allow the strategy to be evaluated against the S&P 500 benchmark using portfolio returns, drawdowns, and risk-adjusted performance metrics.

## Research Question
Can machine learning models trained on lagged 10-Q fundamentals data and historical price features generate predictive buy/sell signals for three-month stock returns?

## Project Preview / Key Results


## Data

## Methodology

## Repository Structure

## Reproducing the Project

## Technologies Used

## Limitations and Future Work

## Data

Raw and cleaned datasets are not included in this repository due to licensing, redistribution, and file-size considerations.

This project used four datasets:

1. [Dataset name]  
   Source: [Yahoo Finance / SEC / other source]  
   Description: [What it contains]

2. [Dataset name]  
   Source: [Source]  
   Description: [What it contains]

3. [Dataset name]  
   Source: [Source]  
   Description: [What it contains]

4. [Dataset name]  
   Source: [Source]  
   Description: [What it contains]

Expected local structure:

- `data/raw/`: original source files
- `data/clean/`: processed datasets generated during the project

To reproduce the project, obtain the source data locally and place it in the expected folders.