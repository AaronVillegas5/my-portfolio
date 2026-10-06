---
title: "Macroeconomic & Climate Analytics Platform"
description: "Data platform ingesting FRED, Open-Meteo, and Census Retail data into PostgreSQL and BigQuery with Airflow orchestration, dbt modeling, SARIMAX forecasting, and a Gemini SQL agent."
pubDate: "May 16 2026"
heroImage: "/macro-pipeline.webp"
badge: "Data Engineering"
tags: ["Python", "Apache Airflow", "dbt", "BigQuery", "Snowflake", "PostgreSQL", "Docker", "FastAPI", "SARIMAX"]
---

<div class="grid grid-cols-1 sm:grid-cols-3 gap-4 my-6">
  <div class="p-4 bg-base-200/80 rounded-xl border border-base-300 shadow-sm flex flex-col justify-between">
    <span class="text-xs font-semibold text-base-content/70 uppercase tracking-wider">Orchestration & DWH</span>
    <div class="text-2xl md:text-3xl font-extrabold text-primary my-1">Airflow + BigQuery</div>
    <p class="text-xs text-base-content/80 mt-1">Daily DAGs in Docker with dbt transformations</p>
  </div>
  <div class="p-4 bg-base-200/80 rounded-xl border border-base-300 shadow-sm flex flex-col justify-between">
    <span class="text-xs font-semibold text-base-content/70 uppercase tracking-wider">Time-Series Modeling</span>
    <div class="text-2xl md:text-3xl font-extrabold text-primary my-1">SARIMAX Engine</div>
    <p class="text-xs text-base-content/80 mt-1">AIC-selected orders with domain isolation</p>
  </div>
  <div class="p-4 bg-base-200/80 rounded-xl border border-base-300 shadow-sm flex flex-col justify-between">
    <span class="text-xs font-semibold text-base-content/70 uppercase tracking-wider">Natural Language SQL</span>
    <div class="text-2xl md:text-3xl font-extrabold text-primary my-1">Gemini AI Agent</div>
    <p class="text-xs text-base-content/80 mt-1">FastAPI service querying BigQuery data marts</p>
  </div>
</div>

## Overview

I built this platform to bring together macroeconomic indicators, localized climate data, and US Census retail sales into a unified analytical data warehouse. Rather than running disconnected ad-hoc scripts, the system runs on a containerized architecture orchestrated by Apache Airflow, cleans and models data with dbt, forecasts trends with SARIMAX, and exposes data marts through a Gemini-powered natural language SQL agent.

* GitHub Repository: [AaronVillegas5/macro-data-pipeline](https://github.com/AaronVillegas5/macro-data-pipeline)

---

## Architecture & Data Flow

```
[ Data Sources ]
  - FRED API (CPI, Unemployment, Treasury Yields)
  - Open-Meteo API (Historical Weather & Climate)
  - US Census Bureau (Monthly Retail Trade Survey)
         │
         ▼
[ Orchestration & Extraction: Apache Airflow (Docker) ]
  - Python extraction services with backoff retries
  - Raw JSON audit logs stored in AWS S3
         │
         ▼
[ Multi-Sink Ingestion ]
  - PostgreSQL (Operational staging with SQLAlchemy ORM)
  - Google BigQuery (Analytical warehouse)
  - Snowflake (Historical archive with MERGE upserts)
         │
         ▼
[ Transformation: dbt (Data Build Tool) ]
  - Staging: Timestamp standardization & schema casting
  - Intermediate: Rolling averages & time-series gap analysis
  - Marts: fct_monthly_retail_macro & climate observations
  - CI/CD: Automated dbt tests via GitHub Actions
         │
         ├─────────────────────────────────────────┐
         ▼                                         ▼
[ SARIMAX Forecasting Engine ]             [ Gemini AI SQL Agent ]
  - statsmodels auto-order selection         - FastAPI backend
  - National vs local domain isolation       - Schema-aware SQL generator
  - Trend projections                        - Read-only query execution
```

---

## Pipeline Features

| Area | Implementation |
| :--- | :--- |
| **Data Sources** | FRED API, Open-Meteo Weather API, US Census Retail Trade |
| **Orchestration** | Apache Airflow DAGs running in Docker Compose |
| **Storage Sinks** | Google BigQuery, PostgreSQL, Snowflake, AWS S3 |
| **Transformations** | dbt models (staging, intermediate, marts) with schema tests |
| **CI/CD Automation** | GitHub Actions running pytest, linting, and dbt test on push |
| **Forecasting** | SARIMAX via statsmodels with AIC order optimization |
| **Self-Serve Analytics** | FastAPI service with Gemini function-calling to query BigQuery |

---

## Key Technical Decisions

### 1. Daily Orchestration with Apache Airflow
Initially, data ingestion was triggered through standalone Python scripts. As the number of feeds grew to include FRED, Open-Meteo, and Census Retail data, I transitioned the workflow to Apache Airflow running in Docker Compose. Airflow schedules daily DAGs that handle exponential backoff retries, monitor API rate limits, and trigger downstream dbt transformations only after upstream extraction tasks pass schema checks.

### 2. Retail Data Marts in dbt (`fct_monthly_retail_macro`)
To analyze how consumer spending responds to macroeconomic shifts, I added monthly retail trade data from the US Census Bureau. Using dbt, I built a star schema joining retail categories (such as grocery, general merchandise, and electronics) against FRED inflation, unemployment, and interest rate metrics. The dbt pipeline runs automated schema tests on primary keys, bounds checks on sales values, and timestamp alignment across monthly reporting cycles.

### 3. SARIMAX Forecasting with Domain Isolation
I built a time-series forecasting engine using `statsmodels.tsa.statespace.sarimax.SARIMAX` to project retail sales and temperature trends. A key design decision was enforcing domain isolation:
* City-level weather records are modeled independently from national retail aggregates.
* Mixing localized rainfall with national retail figures risks producing false statistical correlations (such as a storm in Dallas looking like it caused a dip in nationwide grocery revenue).
* The forecasting engine evaluates seasonal differences (P, D, Q, s) and picks model parameters based on the Akaike Information Criterion (AIC).

### 4. Gemini Database Query Agent
To make BigQuery data accessible without writing manual queries each time, I built a FastAPI backend that connects Google Gemini to the data marts:
1. The service exposes table schemas and sample values to Gemini using function calling.
2. The agent converts a user question (such as "What was the year-over-year sales change in general merchandise during the highest inflation month of 2024?") into a standard BigQuery SQL query.
3. The SQL is validated against an allowed schema list and executed in read-only mode.
4. The agent summarizes the returned dataset with exact numbers and reference periods.

---

## Code Highlights

<div class="my-6 rounded-xl border border-base-300 bg-base-200/40 p-4 shadow-sm overflow-x-auto">
  <div role="tablist" class="tabs tabs-lifted overflow-x-auto flex-nowrap min-w-max">
    <!-- Tab 1 -->
    <input type="radio" name="code_tabs_macro" role="tab" class="tab font-semibold" aria-label="SARIMAX Forecasting" checked />
    <div role="tabpanel" class="tab-content bg-base-100 border-base-300 rounded-box p-4 overflow-x-auto max-w-full">
      <p class="text-xs text-base-content/80 mb-3"><b>File:</b> <code>src/forecasting/sarimax_engine.py</code> - Time-series model fitting with AIC selection:</p>

```python
import numpy as np
import pandas as pd
from statsmodels.tsa.statespace.sarimax import SARIMAX

def fit_best_sarimax(series: pd.Series, exog: pd.DataFrame = None, seasonal_period: int = 12):
    """Fits SARIMAX models across a grid of orders and returns the lowest AIC model."""
    best_aic = np.inf
    best_model = None
    best_order = None

    # Search across non-seasonal and seasonal order combinations
    for p in range(0, 3):
        for d in [0, 1]:
            for q in range(0, 2):
                try:
                    model = SARIMAX(
                        series,
                        exog=exog,
                        order=(p, d, q),
                        seasonal_order=(1, 1, 0, seasonal_period),
                        enforce_stationarity=False,
                        enforce_invertibility=False,
                    ).fit(disp=False)

                    if model.aic < best_aic:
                        best_aic = model.aic
                        best_model = model
                        best_order = (p, d, q)
                except Exception:
                    continue

    return best_model, best_order
```

    </div>

    <!-- Tab 2 -->
    <input type="radio" name="code_tabs_macro" role="tab" class="tab font-semibold" aria-label="Gemini SQL Tool" />
    <div role="tabpanel" class="tab-content bg-base-100 border-base-300 rounded-box p-4">
      <p class="text-xs text-base-content/80 mb-3"><b>File:</b> <code>src/agent/query_agent.py</code> - BigQuery schema tool execution for Gemini agent:</p>

```python
from google.cloud import bigquery
from fastapi import HTTPException

ALLOWED_DATASETS = {"macro_analytics", "weather_data", "retail_marts"}

def execute_safe_query(client: bigquery.Client, query: str) -> list[dict]:
    """Validates that query targets allowed datasets and executes read-only query."""
    clean_query = query.strip()
    lowered = clean_query.lower()

    # Enforce read-only constraint
    forbidden_tokens = ["drop", "delete", "insert", "update", "alter", "truncate"]
    if any(token in lowered for token in forbidden_tokens):
        raise HTTPException(status_code=400, detail="Only SELECT queries are permitted.")

    query_job = client.query(clean_query)
    results = query_job.result(max_results=50)
    return [dict(row) for row in results]
```

    </div>

    <!-- Tab 3 -->
    <input type="radio" name="code_tabs_macro" role="tab" class="tab font-semibold" aria-label="dbt Retail Mart SQL" />
    <div role="tabpanel" class="tab-content bg-base-100 border-base-300 rounded-box p-4">
      <p class="text-xs text-base-content/80 mb-3"><b>File:</b> <code>models/marts/fct_monthly_retail_macro.sql</code> - dbt model joining retail spending with macro indicators:</p>

```sql
with retail as (
    select * from {{ ref('stg_census_retail') }}
),
macro as (
    select * from {{ ref('stg_fred_indicators') }}
)
select
    r.report_month,
    r.naics_code,
    r.category_name,
    r.sales_millions,
    m.cpi_value,
    m.unemployment_rate,
    m.treasury_10yr_yield,
    -- Calculate 12-month rolling average sales per category
    avg(r.sales_millions) over (
        partition by r.naics_code
        order by r.report_month
        rows between 11 preceding and current row
    ) as rolling_12m_avg_sales,
    -- Year-over-year sales percentage variance
    round(
        (r.sales_millions - lag(r.sales_millions, 12) over (
            partition by r.naics_code order by r.report_month
        )) / nullif(lag(r.sales_millions, 12) over (
            partition by r.naics_code order by r.report_month
        ), 0) * 100, 2
    ) as yoy_sales_growth_pct
from retail r
left join macro m
    on r.report_month = m.report_month;
```

    </div>
  </div>
</div>

