---
title: "Macroeconomic Time-Series Data Pipeline"
description: "Multi-source time-series ETL pipeline ingesting macroeconomic and weather APIs into PostgreSQL, BigQuery, and Snowflake with AWS S3 raw logging."
pubDate: "May 16 2026"
heroImage: "/macro-pipeline.webp"
badge: "Data Engineering"
tags: ["Python", "PostgreSQL", "BigQuery", "Snowflake", "AWS S3", "REST APIs"]
---

<!-- 📊 KPI Summary Grid -->
<div class="grid grid-cols-1 sm:grid-cols-3 gap-4 my-6">
  <div class="p-4 bg-base-200/80 rounded-xl border border-base-300 shadow-sm flex flex-col justify-between">
    <span class="text-xs font-semibold text-base-content/70 uppercase tracking-wider">Cloud & DWH Architecture</span>
    <div class="text-2xl md:text-3xl font-extrabold text-primary my-1">3 Storage Sinks</div>
    <p class="text-xs text-base-content/80 mt-1">PostgreSQL, BigQuery, Snowflake</p>
  </div>
  <div class="p-4 bg-base-200/80 rounded-xl border border-base-300 shadow-sm flex flex-col justify-between">
    <span class="text-xs font-semibold text-base-content/70 uppercase tracking-wider">Data Deduplication</span>
    <div class="text-2xl md:text-3xl font-extrabold text-primary my-1">100% Idempotent</div>
    <p class="text-xs text-base-content/80 mt-1">ON CONFLICT & MERGE INTO Upserts</p>
  </div>
  <div class="p-4 bg-base-200/80 rounded-xl border border-base-300 shadow-sm flex flex-col justify-between">
    <span class="text-xs font-semibold text-base-content/70 uppercase tracking-wider">API Persistence</span>
    <div class="text-2xl md:text-3xl font-extrabold text-primary my-1">Dual-Sink</div>
    <p class="text-xs text-base-content/80 mt-1">AWS S3 Raw Logging + Streaming Ingestion</p>
  </div>
</div>

## Executive Summary

Economic analysis and time-series forecasting rely heavily on consistent, reliable, and multi-source data ingestion. The **Macro Data Pipeline** is an automated data engineering system designed to collect, clean, deduplicate, and store historical economic and environmental datasets from multiple REST APIs (Federal Reserve Economic Data - FRED API, Open-Meteo Weather API) into a unified multi-cloud data architecture.

The pipeline architecture features dual-sink streaming to **PostgreSQL** and **Google BigQuery**, analytical data warehousing in **Snowflake**, and raw API JSON response persistence in **AWS S3** for auditability and reprocessing.

---

## 🖥 Architecture & Infrastructure Diagram

The pipeline isolates each data source using modular service handlers and non-blocking try/except sinks, ensuring a failure in one destination (e.g. cloud latency) never blocks ingestion into another.

<div class="my-6 p-4 rounded-xl border border-base-300 bg-base-200/60 shadow-sm overflow-x-auto font-mono text-xs md:text-sm text-base-content">
<pre class="leading-relaxed">
┌────────────────────────────────────────────────────────────────────────┐
│                   API Ingestion Layer (FRED & Open-Meteo)              │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│               Data Fetchers & Parser Services (Python)                 │
└──────────────┬────────────────────┬────────────────────┬───────────────┘
               │                    │                    │
               ▼                    ▼                    ▼
     ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
     │   AWS S3 Raw     │  │ PostgreSQL DWH   │  │  Google BigQuery │
     │ (JSON Data Lake) │  │  (SQLAlchemy)    │  │  (Streaming Sink)│
     └──────────────────┘  └──────────────────┘  └──────────────────┘
                                    │
                                    ▼
                           ┌──────────────────┐
                           │  Snowflake DWH   │
                           │  (MERGE Upsert)  │
                           └──────────────────┘
</pre>
</div>

* 💻 **GitHub Repository:** [AaronVillegas5/macro-data-pipeline](https://github.com/AaronVillegas5/macro-data-pipeline)

---

## 📊 Key Pipeline Features & Specifications

| Feature / Metric | Pipeline Implementation |
| :--- | :--- |
| **Data Sources** | FRED API (Federal Reserve), Open-Meteo Historical Weather API, U.S. Census |
| **Relational Database** | PostgreSQL (managed with Alembic schema migrations) |
| **Cloud Warehouses** | Google BigQuery (weather_data dataset) & Snowflake (analytical DWH) |
| **Raw Persistence** | AWS S3 Bucket (S3 Object Storage for raw API responses) |
| **Migration & Backfill** | Keyset pagination script (`migrate_historical_weather.py`) PostgreSQL → BigQuery |
| **Deduplication** | Idempotent `MERGE INTO` SQL upserts and timestamp uniqueness constraints |

---

## 💡 Economic Insights & Analytical Capabilities

By linking macro indicators with localized weather trends and consumer retail patterns, this pipeline allows analysts to explore:

* **Macroeconomic Inflation & Yield Spreads:** Historical FRED indicator trends (CPI, Unemployment Rate, 10-Year Treasury Yields).
* **Climate & Weather Aggregations:** 30-day rolling average temperature trends, regional rainiest years, and historical extreme weather shifts.
* **Store & Regional Sensitivity:** Correlation between localized weather events and regional retail purchasing patterns.

---

## 🛠 Interactive Code Highlights & Technical Implementation

<div class="my-6 rounded-xl border border-base-300 bg-base-200/40 p-4 shadow-sm overflow-x-auto">
  <div role="tablist" class="tabs tabs-lifted overflow-x-auto flex-nowrap min-w-max">
    <!-- Tab 1 -->
    <input type="radio" name="code_tabs_macro" role="tab" class="tab font-semibold" aria-label="Python Keyset Migration" checked />
    <div role="tabpanel" class="tab-content bg-base-100 border-base-300 rounded-box p-4 overflow-x-auto max-w-full">
      <p class="text-xs text-base-content/80 mb-3"><b>File:</b> <code>scripts/migrate_historical_weather.py</code> — Keyset streaming handler for BigQuery backfill:</p>

```python
from google.cloud import bigquery
from sqlalchemy.orm import Session

_BQ_DATASET = "weather_data"
_BQ_TABLE = "observations_v2"

def _observation_to_bq_row(obs: WeatherObservation) -> dict:
    """Serializes a WeatherObservation ORM instance into a BigQuery-safe dict."""
    return {
        "location_id": obs.location_id,
        "temperature_c": obs.temperature_c,
        "wind_speed": obs.wind_speed,
        "pressure": obs.pressure,
        "humidity": obs.humidity,
        "observed_at": obs.observed_at.isoformat() if obs.observed_at else None,
        "created_at": obs.created_at.isoformat() if obs.created_at else None,
    }

def _stream_batch(client: bigquery.Client, table_ref: str, rows: list[dict], dry_run: bool) -> int:
    """Stream a list of row dicts to BigQuery."""
    if dry_run:
        logger.info("[DRY-RUN] Would stream %d rows to %s", len(rows), table_ref)
        return 0

    errors = client.insert_rows_json(table_ref, rows)
    if errors:
        logger.error("BigQuery insert errors in batch: %s", errors)
        return len(errors)
    return 0
```

    </div>

    <!-- Tab 2 -->
    <input type="radio" name="code_tabs_macro" role="tab" class="tab font-semibold" aria-label="BigQuery Sub-Zero Streak" />
    <div role="tabpanel" class="tab-content bg-base-100 border-base-300 rounded-box p-4">
      <p class="text-xs text-base-content/80 mb-3"><b>File:</b> <code>sql/bigquery/longest_subzero_streak.sql</code> — Gaps & Islands consecutive freeze streak query:</p>

```sql
-- Gaps and Islands: Identify longest consecutive streaks of sub-zero temperature days
WITH DailyTemperatures AS (
    SELECT 
        location_id,
        DATE(observed_at) AS observation_date,
        AVG(temperature_c) AS daily_temp
    FROM `weather_data.observations_v2`
    GROUP BY location_id, DATE(observed_at)
    HAVING AVG(temperature_c) < 0
),
StreakGroups AS (
    SELECT 
        location_id,
        observation_date,
        daily_temp,
        -- Group consecutive dates using difference of row numbers
        observation_date - INTERVAL ROW_NUMBER() OVER (PARTITION BY location_id ORDER BY observation_date) DAY AS streak_group
    FROM DailyTemperatures
)
SELECT 
    location_id,
    MIN(observation_date) AS streak_start,
    MAX(observation_date) AS streak_end,
    COUNT(*) AS consecutive_subzero_days,
    ROUND(AVG(daily_temp), 2) AS avg_streak_temp
FROM StreakGroups
GROUP BY location_id, streak_group
ORDER BY consecutive_subzero_days DESC;
```

    </div>

    <!-- Tab 3 -->
    <input type="radio" name="code_tabs_macro" role="tab" class="tab font-semibold" aria-label="PostgreSQL Upsert" />
    <div role="tabpanel" class="tab-content bg-base-100 border-base-300 rounded-box p-4">
      <p class="text-xs text-base-content/80 mb-3"><b>File:</b> <code>sql/postgres/upsert_weather.sql</code> — Idempotent <code>ON CONFLICT</code> PostgreSQL deduplication:</p>

```sql
INSERT INTO weather_observations (location_id, observed_at, temperature_c, humidity, created_at)
VALUES (:location_id, :observed_at, :temperature_c, :humidity, NOW())
ON CONFLICT (location_id, observed_at) 
DO UPDATE SET 
    temperature_c = EXCLUDED.temperature_c,
    humidity = EXCLUDED.humidity,
    updated_at = NOW();
```

    </div>
  </div>
</div>
