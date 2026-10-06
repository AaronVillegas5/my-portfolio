---
title: "Macroeconomic & Climate Analytics Platform"
description: "Data platform ingesting FRED, Open-Meteo, and Census Retail data into PostgreSQL and BigQuery with Airflow orchestration, dbt modeling, SARIMAX forecasting, and a Gemini data agent."
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
    <span class="text-xs font-semibold text-base-content/70 uppercase tracking-wider">Natural Language Questions</span>
    <div class="text-2xl md:text-3xl font-extrabold text-primary my-1">Gemini AI Agent</div>
    <p class="text-xs text-base-content/80 mt-1">FastAPI service querying BigQuery data marts</p>
  </div>
</div>

## Overview

I built this platform to bring together macroeconomic indicators, localized climate data, and US Census retail sales into a unified analytical data warehouse. Rather than running disconnected ad-hoc scripts, the system runs on a containerized architecture orchestrated by Apache Airflow, cleans and models data with dbt, forecasts trends with SARIMAX, and exposes data marts through a Gemini-powered agent that answers plain-English questions through a small set of query tools.

* GitHub Repository: [AaronVillegas5/macro-data-pipeline](https://github.com/AaronVillegas5/macro-data-pipeline)

---

## Architecture & Data Flow

```
[ Data Sources ]
  - FRED API (CPI, Unemployment, Fed Funds Rate, GDP, Sentiment)
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
[ SARIMAX Forecasting Engine ]             [ Gemini Data Agent ]
  - statsmodels auto-order selection         - FastAPI backend
  - National vs local domain isolation       - Four fixed BigQuery query tools
  - Trend projections                        - SELECT-only queries (read-only)
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
To analyze how consumer spending responds to macroeconomic shifts, I added monthly retail trade data from the US Census Bureau. Using dbt, I pivoted the retail series (total, grocery, e-commerce, auto, and clothing sales) and joined it onto the existing macro and weather fact table, so one mart answers questions across all three sources. The dbt project also runs automated tests, including positive-sales checks, no-future-dates checks, and out-of-bounds checks.

### 3. SARIMAX Forecasting with Domain Isolation
I built a time-series forecasting engine using `statsmodels.tsa.statespace.sarimax.SARIMAX` to project retail sales and temperature trends. A key design decision was enforcing domain isolation:
* City-level weather records are modeled independently from national retail aggregates.
* Mixing localized rainfall with national retail figures risks producing false statistical correlations (such as a storm in Dallas looking like it caused a dip in nationwide grocery revenue).
* The forecasting engine evaluates seasonal differences (P, D, Q, s) and picks model parameters based on the Akaike Information Criterion (AIC).

### 4. Gemini Data Agent
To make the BigQuery data accessible without writing manual queries each time, I built a FastAPI service that connects Google Gemini to the data marts through function calling:
1. The agent is given four fixed query functions: a weather and macro mart query, a data freshness check, a climate extremes lookup, and a city-to-city comparison.
2. Given a plain-English question, Gemini decides which function to call and with what arguments (dates, city, metric).
3. Each function runs a parameterized SELECT-only query against the mart. The agent does not write free-form SQL.
4. The agent summarizes the returned rows with exact numbers and reference periods.

---

## Code Highlights

<div class="my-6 rounded-xl border border-base-300 bg-base-200/40 p-4 shadow-sm overflow-x-auto">
  <div role="tablist" class="tabs tabs-lifted overflow-x-auto flex-nowrap min-w-max">
    <!-- Tab 1 -->
    <input type="radio" name="code_tabs_macro" role="tab" class="tab font-semibold" aria-label="SARIMAX Forecasting" checked />
    <div role="tabpanel" class="tab-content bg-base-100 border-base-300 rounded-box p-4 overflow-x-auto max-w-full">
      <p class="text-xs text-base-content/80 mb-3"><b>File:</b> <code>services/forecasting/sarimax_engine.py</code> - AIC grid search over seasonal and non-seasonal orders (excerpt):</p>

```python
def _aic_grid_search(
    train: pd.Series,
    p_values: tuple[int, ...] = (0, 1, 2),
    d_values: tuple[int, ...] = (0, 1),
    q_values: tuple[int, ...] = (0, 1, 2),
    P_values: tuple[int, ...] = (0, 1),
    D_values: tuple[int, ...] = (0, 1),
    Q_values: tuple[int, ...] = (0, 1),
    s: int = 12,
) -> tuple[object, tuple, tuple, float]:
    """Lightweight grid search over (p,d,q)(P,D,Q)_s minimising AIC."""
    best_result = None
    best_order = (1, 1, 1)
    best_seasonal_order = (1, 1, 1, s)
    best_aic = float("inf")

    for order in itertools.product(p_values, d_values, q_values):
        for seasonal_order_pdq in itertools.product(P_values, D_values, Q_values):
            seasonal_order = (*seasonal_order_pdq, s)
            result = _try_fit(train, order, seasonal_order)
            if result is not None and result.aic < best_aic:
                best_aic = result.aic
                best_result = result
                best_order = order
                best_seasonal_order = seasonal_order
    # ... falls back to ARIMA(1,1,1) if no model converges
```

    </div>

    <!-- Tab 2 -->
    <input type="radio" name="code_tabs_macro" role="tab" class="tab font-semibold" aria-label="Gemini Agent Tools" />
    <div role="tabpanel" class="tab-content bg-base-100 border-base-300 rounded-box p-4">
      <p class="text-xs text-base-content/80 mb-3"><b>File:</b> <code>services/ai_agent.py</code> - the Gemini agent is given four fixed BigQuery query functions to call (excerpt):</p>

```python
config = types.GenerateContentConfig(
    tools=[
        query_macro_weather_mart,
        check_data_freshness,
        get_climate_extremes,
        compare_city_climates,
    ],
    temperature=0.2,
    system_instruction=(
        "You are an expert Macroeconomic & Climate Research Analyst. "
        "Always use your tools to query the official BigQuery data marts and database before answering. "
        # ...
        "Provide executive summaries with key statistics and trends."
    ),
)
```

    </div>

    <!-- Tab 3 -->
    <input type="radio" name="code_tabs_macro" role="tab" class="tab font-semibold" aria-label="dbt Retail Mart SQL" />
    <div role="tabpanel" class="tab-content bg-base-100 border-base-300 rounded-box p-4">
      <p class="text-xs text-base-content/80 mb-3"><b>File:</b> <code>dbt/models/marts/fct_monthly_retail_macro.sql</code> - dbt model joining monthly retail sales onto the macro and weather fact table:</p>

```sql
SELECT
    w.*,
    r.total_retail_sales_millions,
    r.total_retail_sales_yoy_growth_pct,
    r.grocery_sales_millions,
    r.ecommerce_sales_millions,
    r.auto_sales_millions,
    r.clothing_sales_millions
FROM {{ ref('fct_monthly_macro_weather') }} w
LEFT JOIN {{ ref('int_retail_pivoted') }} r
ON
    w.year_month = FORMAT('%d-%02d', r.observed_year, r.observed_month)
```

    </div>
  </div>
</div>
