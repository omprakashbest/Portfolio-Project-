# Customer Churn Analysis

An end-to-end customer churn analysis project for an OTT-style subscription business. The project combines customer, subscription, and support data from a SQLite database, prepares an analysis-ready customer table, and examines churn, revenue exposure, contracts, plans, and support signals.

## Highlights

- 21 customer records across three relational source tables
- 28.6% overall churn rate (6 churned customers)
- 60.0% churn for the Basic plan
- Rs 73.94 in monthly charges associated with churned customers
- 0.77 escalation-to-churn correlation

> This is a small portfolio dataset. The findings are descriptive and should not be treated as statistically conclusive.

## Project structure

```text
.
|- Data/
|  |- customer_churn.db               # SQLite source database
|  `- customer_churn_data_raw.xlsx    # Raw-table reference copy
|- churn_analysis.ipynb               # Cleaning, analysis, and charts
`- exported_churn_data.csv            # Customer-level analysis dataset
```

## Data model

The SQLite database contains three tables joined by `customerid`:

| Table | Grain | Key fields |
| --- | --- | --- |
| `db_customer` | One row per customer | demographics, location, date of birth |
| `db_subscription` | One row per customer subscription | plan, contract, charges, CLTV, cancellation, churn score |
| `db_support` | One row per support interaction | complaint date, escalation, CSAT |

## Data preparation

The notebook performs the following transformations:

1. Renames `name` to `customer_name` and removes unused customer attributes.
2. Standardizes gender labels and fills missing country values using the state-to-country mapping present in the data.
3. Converts date fields to date types.
4. Creates `churn_Flag`: `1` where a cancellation date exists, otherwise `0`.
5. Resolves repeated support interactions by retaining the latest interaction per customer and adding `complaint_count`.
6. Merges the three tables to produce `exported_churn_data.csv`.

## Analysis questions

- What are the overall churn and retention rates?
- Which plan types and states have the highest churn?
- How much monthly revenue is associated with churned customers?
- How does tenure differ across the customer base?
- Are support escalations associated with churn?
- Which customers fall into low, medium, and high churn-risk segments?

## Key findings

- Six of 21 customers churned, yielding a churn rate of **28.6%**.
- Churned customers account for **Rs 73.94** in monthly charges.
- Cancellation activity peaks in September 2024 within this sample.
- The Basic plan has the highest churn rate at **60.0%**.

## Notebook-backed KPIs

The table below lists only KPIs found in `churn_analysis.ipynb`. Values reflect the notebook's saved output or its direct calculation against `exported_churn_data.csv`.

| KPI | Notebook calculation | Verified value | Note |
| --- | --- | ---: | --- |
| Churn rate | Mean of `churn_Flag` | 28.57% | `churn_Flag` is 1 when `cancellation_date` is present. |
| Retention rate | `100 - churn rate` | 71.43% | Direct complement of churn rate. |
| Churn by plan type | Mean `churn_Flag`, grouped by `plan_type` | Basic 60.00%; Standard 22.22%; Premium 14.29% | Found in the notebook. |
| Churn by state | Mean `churn_Flag`, grouped by `state` | See state table below | The notebook groups by state, not country and state together. |
| ARPU | Mean of `monthly_charges` | Rs 18.85 | The notebook includes all customers, not only active customers. |
| Average customer tenure | Average of cancellation/start days, or current date/start days for active customers | 1,551 days* | Uses the current date at run time. |
| Revenue at risk | Sum `monthly_charges` where `churn_Flag = 1` | Rs 73.94 | This differs from the report formula based on `churn_score > 70`. |
| Escalation rate | Share of all customers with `escalations = 'Y'` | 19.05% | This differs from `SUM(escalations) / COUNT(complaints)`. |
| Average complaints per customer | Sum `complaint_count` / unique customers | 0.43 | Support interactions are deduplicated before the customer-level merge. |
| Escalation-to-churn correlation | Pearson correlation of encoded escalation and `churn_Flag` | 0.77 | Association only; it does not establish causation. |

\*Tenure is time-sensitive because active-customer tenure uses the current date when the notebook runs. The displayed figure is from the notebook's saved execution.

### Churn by state

| State | Customers | Churned customers | Churn rate |
| --- | ---: | ---: | ---: |
| Karnataka | 2 | 2 | 100.00% |
| Meghalaya | 3 | 2 | 66.67% |
| Telangana | 2 | 1 | 50.00% |
| Delhi | 4 | 1 | 25.00% |
| Kathmandu | 1 | 0 | 0.00% |
| Maharashtra | 3 | 0 | 0.00% |
| Nagaland | 2 | 0 | 0.00% |
| Rajasthan | 2 | 0 | 0.00% |
| Uttar Pradesh | 2 | 0 | 0.00% |

## Verified insights

- **Churn and retention:** 6 of 21 customers churned, producing a **28.57% churn rate** and **71.43% retention rate**.
- **Plan risk:** The Basic plan has the highest churn rate at **60.00%** (3 of 5 customers). This identifies a segment to investigate; the notebook does not calculate plan-level revenue impact.
- **Cancellation timing:** September 2024 has the most cancellations in the sample, with **2 churned customers**.
- **State view:** Karnataka has a **100.00% churn rate** (2 of 2 customers). Meghalaya also has 2 churned customers, but its rate is **66.67%** (2 of 3). The dataset is too small for a conclusive geographic claim.
- **Customer value:** Average monthly charge is **Rs 18.85**. The notebook identifies **Rs 73.94** in monthly charges associated with churned customers.


The notebook contains the project charts, including monthly churn trend, churn by plan type, churn by state, and correlation visualizations. See [churn_analysis.ipynb](churn_analysis.ipynb) for the generated figures.

## Getting started

### Prerequisites

- Python 3.10+
- Jupyter Notebook or JupyterLab

Install the libraries used by the notebook:

```bash
pip install pandas numpy matplotlib seaborn
```

### Run the analysis

1. Clone the repository.
2. Open `churn_analysis.ipynb` in Jupyter.
3. In the database connection cell, update the SQLite path if your local clone is in a different location.
4. Run the notebook from top to bottom.

The notebook reads `Data/customer_churn.db` and regenerates `exported_churn_data.csv`.

## Notes and limitations

- The raw support table contains multiple interactions for two customers. The analysis preserves the latest interaction and separately counts total complaints.
- Customer demographics contain fields such as names and dates of birth. Treat the repository as a portfolio example and do not use those fields for real customer outreach without appropriate governance.
- Current churn is defined solely by whether `cancellation_date` is populated.
- The small sample size makes segment-level comparisons directional rather than definitive.

## Tools used

- Python: pandas, NumPy, Matplotlib, Seaborn
- SQLite
- Jupyter Notebook
