# retail-data-cleaning-pipeline

A cleaning pipeline for the UCI Online Retail II dataset, built as my Apex Project for the BS in
Data Science & AI at BITS Pilani (Trimester 3, problem P4: Retail and E-Commerce).

The raw data is 1,067,371 transactions from a UK online gift retailer, December 2009 to December
2011, and it is messy in the ways real sales data usually is: returns mixed in with sales as
negative quantities, missing customer IDs on roughly a quarter of the rows, the same product
spelled several ways, prices of zero, and duplicate invoice lines. An inventory assistant is only
as good as the data behind it, so the pipeline turns this into two clean tables and measures what
every step changed.

Scope: this prepares and analyses the data. No machine learning model is trained here.

## Before and after

| Metric | Before | After |
|---|---|---|
| Rows | 1,067,371 | 1,010,534 |
| Duplicate rows | 34,335 | 0 |
| Negative quantity rows (returns) | 22,950 | 0 |
| Zero or negative prices | 6,207 | 0 |
| Missing descriptions | 4,382 | 0 |
| Distinct StockCodes | 5,305 | 4,984 |
| Missing CustomerID | 243,007 | 231,043 |

The 56,837-row drop is fully explained by the 22,950 returns moved into their own table (kept,
not deleted) and the 33,887 duplicate rows removed from sales. The 231,043 rows still missing a
CustomerID are there on purpose: they are flagged as guests rather than dropped, so product-level
demand keeps those sales. The full table is in `quality_report.csv`.

## The pipeline

| Step | What it does |
|---|---|
| 1 | Load both Excel sheets (2009–2010, 2010–2011) and merge them, keeping the source year |
| 2 | Inspect: types, nulls, duplicates, value counts, recorded as the BEFORE snapshot |
| 3 | Separate the 22,950 returns (negative quantity) from sales; returns are stored, sales go forward |
| 4 | Remove exact duplicate rows |
| 5 | Flag rows with no CustomerID as `is_guest` instead of dropping them |
| 6 | Set zero or negative prices to missing and fill each with the median price for that StockCode, falling back to the overall median where a StockCode has no valid price |
| 7 | Standardise InvoiceDate and derive date, month and year |
| 8 | Standardise StockCode and Description: one canonical description per StockCode, using the most frequent one |
| 9 | Standardise country labels, e.g. `EIRE` to `Ireland`, `RSA` to `South Africa` |
| 10 | Encode categorical columns |
| 11 | Aggregate to one row per product per month: units sold, revenue, number of invoices |
| 12 | Scale units and revenue with MinMaxScaler, keeping the originals alongside |
| 13 | Build the before/after quality report |
| 14 | Export `clean_transactions.csv` and `clean_retail.csv` |

## Analysis

Four charts from the cleaned product-month table, all in the notebook and in `charts/`:

1. Top 20 products by total units sold
2. Zero-sale months per high-volume product, as a rough stockout-risk signal
3. Total monthly revenue, December 2009 to December 2011
4. Top 10 countries by revenue and units sold

![Zero-sale months per high-volume product](charts/2_zero_sale_months.png)

## Outputs

- `clean_transactions.csv`: the cleaned transaction table, 1,010,534 rows
- `clean_retail.csv`: one row per product per month, 67,390 rows
- `quality_report.csv`: the before/after comparison above

The two clean CSVs are not committed because of their size (the transaction table is about
130 MB). Running the notebook regenerates them.

## Run it

1. Download Online Retail II from the UCI Machine Learning Repository:
   https://archive.ics.uci.edu/dataset/502/online+retail+ii
2. Put `online_retail_II.xlsx` next to the notebook.
3. Open `retail_data_cleaning_pipeline.ipynb` in Jupyter or Google Colab and run all cells.

Python 3.10+, pandas, NumPy, scikit-learn, Matplotlib, Seaborn, openpyxl.
