---
title: "Macroeconomic Time-Series Data Pipeline"
description: "Multi-source time-series ETL pipeline ingesting macroeconomic and weather APIs into PostgreSQL, BigQuery, and Snowflake with AWS S3 raw logging."
pubDate: "Apr 18 2026"
heroImage: "/macro-pipeline.webp"
badge: "Data Engineering"
tags: ["Python", "PostgreSQL", "BigQuery", "Snowflake", "AWS S3", "REST APIs"]
---

## Executive Summary

Economic analysis and time-series forecasting rely heavily on consistent, reliable, and multi-source data ingestion. The **Macro Data Pipeline** is an automated data engineering system designed to collect, clean, deduplicate, and store historical economic and environmental datasets from multiple REST APIs (Federal Reserve Economic Data - FRED API, Open-Meteo Weather API) into a unified multi-cloud data architecture.

The pipeline architecture features dual-sink streaming to **PostgreSQL** and **Google BigQuery**, analytical data warehousing in **Snowflake**, and raw API JSON response persistence in **AWS S3** for auditability and reprocessing.

---

## 🖥 Architecture & Infrastructure Diagram

The pipeline isolates each data source using modular service handlers and non-blocking try/except sinks, ensuring a failure in one destination (e.g. cloud latency) never blocks ingestion into another.

```
API Layer (FRED, Open-Meteo)
        │
        ▼
  Data Fetchers
        │
        ├──▶ Raw JSON  ──▶ AWS S3 (Audit / Raw Data Lake)
        │
        └──▶ Parsers / Cleaners
                  │
                  ├──▶ PostgreSQL  (SQLAlchemy MERGE / Upsert)
                  ├──▶ Google BigQuery  (Streaming Insert — Dual Sink)
                  └──▶ Snowflake  (Analytical Data Warehouse)
```

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

## 🛠 Actual Technical Implementation & Code Highlights (From GitHub)

### 1. Keyset Migration & Streaming to BigQuery (`migrate_historical_weather.py`)
Below is the actual implementation of our keyset migration handler streaming `WeatherObservation` ORM rows from PostgreSQL to BigQuery:

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

### 2. Analytical Rolling Window Calculations (BigQuery SQL DDL)
To compute 30-day rolling temperature averages across location clusters efficiently, we run partitioned window queries in BigQuery:

```sql
-- sql/bigquery/avg_temp_rolling_30_days.sql
SELECT 
    location_id,
    observed_at,
    temperature_c,
    AVG(temperature_c) OVER (
        PARTITION BY location_id 
        ORDER BY observed_at 
        ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
    ) AS rolling_30_day_avg_temp
FROM `weather_data.observations_v2`
ORDER BY location_id, observed_at;
```

### 3. PostgreSQL Upsert Deduplication (`sql/postgres/upsert_weather.sql`)
To ensure total idempotency, ingestion jobs execute `ON CONFLICT` updates on location and timestamp keys:

```sql
INSERT INTO weather_observations (location_id, observed_at, temperature_c, humidity, created_at)
VALUES (:location_id, :observed_at, :temperature_c, :humidity, NOW())
ON CONFLICT (location_id, observed_at) 
DO UPDATE SET 
    temperature_c = EXCLUDED.temperature_c,
    humidity = EXCLUDED.humidity,
    updated_at = NOW();
```
