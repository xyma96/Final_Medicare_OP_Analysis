# Medicare Orthotics & Prosthetics Supplier Analysis

**BUSN 32120 — Data Analysis with Python & SQL | Final Project**
Authors: Xiaoyang Ma, Jyun

---

## Project Overview

This project analyzes Medicare fee-for-service claims data for orthotics and prosthetics (O&P) suppliers across the United States. Using the CMS public API and U.S. Census ACS data, we investigate geographic demand patterns, supplier market structure, and the procedure and operational factors that drive Medicare reimbursement.

**Target audience:** Our investment team, evaluating expansion opportunities in the O&P sector. Following the acquisition of an infrastructure company operating in the O&P market, this analysis supports the assessment of whether acquiring independent O&P clinics and consolidating them into a larger platform through a roll-up strategy is viable. By examining service volume, reimbursement patterns, and geographic variation, the project aims to identify high-demand regions, areas of market fragmentation, and potential targets for clinic consolidation.

---

## Repository Contents

| File | Description |
|------|-------------|
| `Final_Medicare_OP_Analysis.ipynb` | Main Python analysis notebook — data retrieval, EDA, feature engineering, modeling |
| `SQL_Queries_Medicare_OP.ipynb` | Standalone SQL notebook — 14 queries with GROUP BY/ROLLUP, JOINs, window functions, subqueries |
| `README.md` | This file |

---

## Data Sources

- **CMS Medicare Physician & Other Practitioners — O&P Suppliers**
  Dataset ID: `1746a83e-bb65-4300-8e02-21edbab77c6b`
  Source: data.cms.gov public API (no API key required)

- **U.S. Census American Community Survey (ACS) 5-Year Estimates**
  Variables: state population (B01003_001E) and median household income (B19013_001E)
  Source: api.census.gov

---

## Key Analyses

### Python Notebook (`Final_Medicare_OP_Analysis.ipynb`)

1. **Data Retrieval** — Paginated API calls to CMS; ~39,700 supplier-HCPCS records across 8,229 unique suppliers
2. **Data Quality Checks** — Missing values, type validation, outlier detection, reimbursement sanity checks, geographic coverage
3. **Exploratory Data Analysis** — 6+ charts (bar, histogram, scatter, heatmap, pairplot, cumulative share); percentile summaries; group-by aggregations by state, supplier, and HCPCS code
4. **External Data Merge** — Census ACS state population and median income joined to O&P data to compute per-capita utilization and payment intensity
5. **Feature Engineering** — Region mapping (`.map()`), payment-to-charge ratios, services-per-beneficiary, log transforms, one-hot encoding of state dummies; dataset expanded from 32 to 91 columns
6. **Predictive Modeling**
   - *Model 1:* OLS Linear Regression — log-payment regressed on log-services, utilization intensity, and Census region dummies (row-level, ~39K records; R², RMSE, MAE, p-values)
   - *Model 2:* Logistic Regression — classify above-median-payment supplier-HCPCS combinations using operational features only, no payment-derived inputs (accuracy, precision, recall, F1, ROC-AUC, confusion matrix)

### SQL Notebook (`SQL_Queries_Medicare_OP.ipynb`)

14 queries in SQLite covering:
- `GROUP BY` with grand-total rollup rows (via `UNION ALL`)
- `INNER JOIN` — O&P data with ACS state table and HCPCS category lookup
- Window functions: `RANK() OVER (PARTITION BY ...)`, `SUM() OVER (ORDER BY ...)`, `LAG()`
- Subqueries (inline) and CTEs (`WITH` clause)

---

## How to Run

1. **Install dependencies:**
   ```bash
   pip install pandas numpy matplotlib seaborn statsmodels scikit-learn requests
   ```

2. **Run `Final_Medicare_OP_Analysis.ipynb`** top-to-bottom in Jupyter.
   - Fetches data live from CMS API (requires internet)
   - Census API call is also live (no key required for basic variables)

3. **Run `SQL_Queries_Medicare_OP.ipynb`** top-to-bottom in Jupyter.
   - Re-fetches the same data and loads it into an in-memory SQLite database
   - All 14 SQL queries execute via `pd.read_sql()`

---

## Main Findings

1. **Geographic Demand Concentration:** Raw service volume is highest in California, Florida, and Texas. When normalized by population, Florida and Arizona exhibit disproportionately high O&P utilization, likely driven by aging demographics and chronic disease prevalence. Demographic-adjusted metrics are more informative than raw volume for identifying attractive expansion markets.

2. **Fragmented Supplier Market:** The supplier landscape follows a Pareto-like distribution — a small number of suppliers dominate service volume within each state, while the majority are small independent providers. This fragmentation is consistent with a market ripe for roll-up and consolidation strategies.

3. **Procedure Mix Drives Revenue:** Most supplier-HCPCS combinations generate modest reimbursements (typically under $200 per service), but a small set of higher-value prosthetic codes accounts for a disproportionate share of total Medicare spending. Procedure mix is therefore a critical determinant of supplier revenue and profitability.

4. **Reimbursement Is Geographically Standardized:** The correlation between state median household income and average Medicare payment is weak (~0.1). Provider revenue potential is driven more by service volume, patient demographics, and procedure mix than by local income levels — meaning reimbursement risk is low across diverse geographic markets.

5. **Scale and Utilization Intensity Predict Higher Payments:** Regression modeling confirms that higher-volume suppliers and those with greater services-per-beneficiary intensity tend to achieve higher average Medicare payments. A logistic classifier using only operational features (no payment inputs) achieves meaningful predictive accuracy for identifying high-payment supplier-procedure combinations, suggesting structural signals useful for due diligence screening.

## Investment Implications

- The fragmented provider landscape presents strong potential for regional consolidation and roll-up strategies.
- Demographic-driven demand markets — particularly states with older or higher-morbidity populations — may offer structurally higher utilization growth.
- Procedure mix management is a key lever for supplier revenue performance and should be a central focus in acquisition diligence.
- Operational scale advantages suggest that larger platforms can achieve higher reimbursement efficiency and stronger financial performance.

## Limitations

- Total payment values were approximated using average payment multiplied by service counts due to CMS API schema limitations, introducing minor aggregation imprecision.
- The analysis does not adjust for patient clinical severity or device complexity, which may influence reimbursement levels.
- The dataset represents a single year of Medicare claims and does not capture multi-year utilization trends or reimbursement changes.
