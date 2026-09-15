# Olist E-Commerce ETL Pipeline

A guided, from-scratch ETL (Extract, Transform, Load) pipeline built on the [Olist Brazilian E-Commerce dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce), built as project in Python and pandas.

The pipeline pulls 9 relational CSV tables live from Kaggle, cleans and investigates each one individually, aggregates the one-to-many tables (`payments`, `reviews`) down to one row per order, and joins everything into a single analysis-ready **fact table at the order-item grain** — one row per item purchased.

📄 **Full write-up:** [`Olist_ETL_Project_Report.docx`](./Olist_ETL_Project_Report.docx) — covers every transformation decision, data-quality finding, and the reasoning behind each fix, step by step.

## What this project does

1. **Extract** — downloads all 9 Olist CSVs directly via the [`kagglehub`](https://github.com/Kaggle/kagglehub) API (no manual file wrangling).
2. **Transform** — for each table: fixes dtypes (real dates, zero-padded zip codes), investigates and documents every null/duplicate pattern found, and resolves them deliberately (fill, aggregate, or leave-and-document).
3. **Load** — joins all 9 tables into one fact table via a sequence of verified left joins, with row-count integrity checked after every single merge, then saves the result as both Parquet and CSV.

## Result

- **`olist_fact_table.parquet`** / **`olist_fact_table.csv`** — 112,650 rows × 40 columns, one row per order item, enriched with full order, product, seller, customer, payment, and review context.

## Key data-quality findings

- 8 orders marked `delivered` with no recorded delivery timestamp (logging gap, not corruption).
- 551 orders had multiple review submissions — resolved by keeping the most recent (tie-broken with full timestamp precision).
- Payments have a true unique key of `(order_id, payment_sequential)`, not `order_id` alone — one order had 29 separate voucher payments.
- 1,000,163 raw geolocation rows collapsed to 19,015 (one per zip code) — 98% were redundant GPS pings.
- One order (`bfbd0f9bdef84302105ad712db648a6c`) is marked delivered, has no payment record, and a 1-star review claiming the product was never received — flagged as an unresolved edge case.

See the full report for the complete list, with reasoning for every decision.

## Tech stack

- Python, pandas
- [`kagglehub`](https://pypi.org/project/kagglehub/) for dataset access
- Jupyter Notebook

## Setup

```bash
pip install -r requirements.txt
```

Run the notebook top to bottom. On first run, `kagglehub.dataset_download(...)` will prompt for Kaggle authentication (browser login, or a `kaggle.json` API token) — see the [kagglehub docs](https://github.com/Kaggle/kagglehub) for details.

## Project status / next steps

This project intentionally skips defensive plumbing (retry logic, schema validation, automated tests) to keep focus on core ETL logic — that robustness is deferred to a future project. `geolocation_clean` is cleaned and ready but not yet joined into the fact table; a natural next step once a specific geographic question (e.g. customer-to-seller distance) calls for it.
