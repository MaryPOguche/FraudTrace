# FraudTrace

A synthetic banking-transaction fraud analytics portfolio project built with
Python, SQL Server (MSSQL), SQL, Jupyter Notebook, and Power BI.

## Project Overview

FraudTrace generates, stores, validates, analyzes, and visualizes
**1,000,000 synthetic banking transactions** to identify suspicious fraud
patterns across **8 African countries**.

### Target scale

| Entity | Count |
|---|---|
| Customers | 10,000 |
| Accounts | 15,000 |
| Devices | 20,000 |
| Locations | 500 |
| Calendar dates | 1,095 (3 years) |
| Transactions | 1,000,000 |

Countries: Nigeria, Kenya, Ghana, South Africa, Egypt, Tanzania, Uganda, Rwanda
(8 currencies: NGN, ZAR, KES, EGP, GHS, TZS, UGX, RWF).

## Tech Stack

| Tool | Purpose |
|------|---------|
| Python 3.14 | Data generation and analysis |
| Faker | Synthetic customer/transaction attributes |
| NumPy | Random distributions for amounts |
| Pandas | Data manipulation |
| pyodbc + SQLAlchemy | MSSQL database connection |
| SQL Server (MSSQL) | Database storage and SQL queries |
| SSMS | Database management |
| Jupyter Notebook | Exploratory data analysis |
| Matplotlib + Seaborn | Charts and visualizations |
| PyMuPDF | Rasterize the Power BI PDF export into preview PNGs |
| Power BI Desktop | Interactive dashboard |

## Project Structure

```
FraudTrace/
|-- data/
|   |-- raw/                  # Generated CSVs (customers, accounts, devices, ...)
|   |-- charts/               # Saved chart images (src/analyze_fraud.py)
|   |-- reports/              # Analysis report
|   |-- exports/              # Power BI CSV exports (src/export_for_powerbi.py)
|-- data_generation/
|   |-- generate_data.py      # Main data generator
|-- sql_queries/
|   |-- 00_create_tables.sql            # Schema DDL (fraudtrace)
|   |-- 00_create_tables_evidence.sql   # DDL proof in throwaway DB (portfolio)
|   |-- 01_load_data.sql
|   |-- 02_validate_data.sql
|   |-- 03_create_indexes.sql
|   |-- 04_business_questions.sql
|   |-- 05_create_views.sql
|-- src/
|   |-- analyze_fraud.py            # Python fraud analysis (charts + report)
|   |-- export_for_powerbi.py       # CSV exports for Power BI
|   |-- rebuild_database.py         # End-to-end rebuild (+ --evidence-only)
|   |-- render_dashboard_preview.py # Static dashboard mockups (target look)
|   |-- export_dashboard_images.py  # Power BI PDF export -> portfolio PNGs
|   |-- make_sample_exports.py      # Small subset exports for fast PB iteration
|   |-- theme.py                    # Shared chart/palette theme
|-- notebooks/
|   |-- eda_fraudtrace.ipynb
|-- dashboard/                # Power BI dashboard (FraudTrace.pbix, guides)
|-- portfolio/                # Portfolio kit, CRISP-DM write-up, evidence, previews
|-- test/                     # pytest suite
|-- .env                      # Database credentials (not committed)
|-- .gitignore
|-- requirements.txt
```

## Star Schema

```
              dim_customers
                   |
    dim_accounts --+-- fact_transactions -- dim_time
                   |
              dim_devices
                   |
             dim_locations
```

## Setup Instructions

### Prerequisites
- Python 3.11+ (developed and tested on 3.14)
- SQL Server (MSSQL) with SSMS
- ODBC Driver 17 for SQL Server
- Power BI Desktop

### 1. Install Python packages
```bash
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
```

### 2. Configure the database connection
Copy the expected values into `.env`:
```
DB_SERVER=localhost\SQLEXPRESS02
DB_DATABASE=fraudtrace
DB_DRIVER={ODBC Driver 17 for SQL Server}
```
Start the SQL Server service if it is not running:
```powershell
Start-Service -Name 'MSSQL$SQLEXPRESS02'
```

### 3. Create the database and tables
In SSMS: `CREATE DATABASE fraudtrace;` then run
`sql_queries/00_create_tables.sql`.

### 4. Generate data
```bash
python data_generation/generate_data.py
```

### 5. Load and prepare the data
Run these in order in SSMS:
`sql_queries/01_load_data.sql` -> `sql_queries/02_validate_data.sql`
-> `sql_queries/03_create_indexes.sql` -> `sql_queries/05_create_views.sql`.

### 6. Run analysis
```bash
python src/analyze_fraud.py
```
Writes charts to `data/charts/` and a summary to
`data/reports/fraud_analysis_summary.txt`.

### 7. Export data for Power BI
```bash
python src/export_for_powerbi.py
```

### 8. Generate validation evidence
```bash
python src/rebuild_database.py --evidence-only
```
Verifies all 12 checks and writes 3 PNGs to `portfolio/evidence/`.

### 9. Open the Jupyter notebook
```bash
jupyter notebook notebooks/eda_fraudtrace.ipynb
```

### 10. Power BI dashboard
The built report is `https://drive.google.com/file/d/1jItrbe6jpiTB3NZ_0L9USvIpCZcQRHm1/view?usp=drive_link` (5 pages, git-ignored). To
rebuild it, import the CSV files from `data/exports/` (or connect directly to
the `vw_*` views) and follow `dashboard/README.md` / `dashboard/BUILD_CHECKLIST.md`.

### 11. Export dashboard previews
In Power BI Desktop use **File -> Export -> Export to PDF**, save the PDF into
`portfolio/`, then rasterize it into the five preview PNGs:
```bash
python src/export_dashboard_images.py --pdf "portfolio/FraudTrace Dashboard Preview.pdf"
```

## Fraud Patterns

The generator injects 6 fraud patterns:

| Pattern | Description |
|---------|-------------|
| OFF_HOURS | Transactions between 2-4 AM with amounts > 10,000 |
| UNUSUAL_AMOUNT | Amounts 15-40x the account average |
| SHARED_DEVICE | Multiple accounts using the same device |
| SALARY_CYCLE | Suspicious activity on salary days (25th-28th) |
| RAPID_TRANSFER | 5+ transfers from one account within 10 minutes |
| IMPOSSIBLE_TRAVEL | Two transactions in different countries within minutes |

## Expected Results

- **Total transactions:** 1,000,000
- **Fraud count:** 29,907 (**fraud rate ~2.99%**)
- **Zero** duplicate transaction IDs
- **Zero** null foreign keys
- **Zero** negative amounts

## SQL Analysis Queries

1. Fraud rate by country
2. High-risk devices
3. Fraud by hour of day
4. Rapid transfers (window function)
5. Average amounts (fraud vs legit)
6. Fraud rate by channel
7. Fraud by transaction type
8. Top 10 fraud accounts
9. Fraud by day of week
10. Weekend vs weekday fraud
11. Fraud by risk profile
12. Fraud by age group
13. Monthly fraud trend
14. Salary day fraud
15. Amount distribution

(All 15 live in `sql_queries/04_business_questions.sql`.)

## Key Findings

- Off-hours transactions (2-4 AM) are extreme fraud hotspots (~47-48% fraud rate).
- Salary days show 6.06% fraud vs 2.51% on other days.
- MOBILE channel leads fraud (Android devices dominate fraud counts).
- Fraud average amount (~9,705) is ~14x the legitimate average (~711).

## Limitations

- All data is synthetic - no real banking data is used
- Fraud patterns are rule-based, not ML-based
- Amounts span 8 currencies, so volume totals blend unlike units and are
  reported without a currency symbol
- Single database instance (not distributed)
- The built Power BI report (`https://drive.google.com/file/d/1jItrbe6jpiTB3NZ_0L9USvIpCZcQRHm1/view?usp=drive_link`) is git-ignored, so it is rebuilt manually in Power BI Desktop

## Power BI Dashboard

The interactive dashboard was developed using Power BI.

[Click here to download the Power BI Dashboard](https://drive.google.com/file/d/1jItrbe6jpiTB3NZ_0L9USvIpCZcQRHm1/view?usp=drive_link)
  

## License

This is a portfolio project for educational purposes. All data is synthetic.
