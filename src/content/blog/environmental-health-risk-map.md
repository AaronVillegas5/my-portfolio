---
title: "Environmental Health Risk Mapping Platform"
description: "Interactive geospatial dashboard predicting asthma and cardiovascular risks across Southern California ZIP codes using XGBoost, Neural Networks, and FastAPI."
pubDate: "May 18 2026"
heroImage: "/env-risk-map.webp"
badge: "Datathon Winner"
tags: ["XGBoost", "FastAPI", "Spatial Analysis", "Python", "Machine Learning"]
---

## Executive Summary

Environmental health risk is not evenly distributed across Southern California. High-risk communities often remain invisible in raw, unaggregated datasets. **Healthy Home Audit** was built during Datathon 2026 to bridge this gap—making complex environmental risk factors accessible, visual, and actionable for residents, policy makers, and health researchers.

Out of 39 competing teams, this platform won **"Best Visualization"** for its interactive mapping capabilities, multi-factorial risk scoring, and real-time machine learning inference engine.

---

## 🖥 Interactive Geospatial Dashboard & Live Demo

The platform visualizes environmental health risk across **8,000+ census tracts and 700+ Southern California ZIP codes**, color-coding markers by burden level (green → red) and allowing users to inspect localized health indicators, demographics, and actionable exposure-reduction tips.

![Environmental Health Risk Heatmap](/env-risk-map.webp)

* 🌐 **Live Web App:** [environmental-health-risk-map.vercel.app](https://environmental-health-risk-map.vercel.app)
* 💻 **GitHub Repository:** [AaronVillegas5/environmental-health-risk-map](https://github.com/AaronVillegas5/environmental-health-risk-map)

---

## 📊 Key Performance Indicators (KPIs) & Coverage

| Metric / Indicator | Project Detail |
| :--- | :--- |
| **Datathon Award** | 🏆 Winner: Best Visualization (1st out of 39 teams) |
| **Geographic Scope** | 700+ ZIP codes / 8,000+ Census Tracts (Southern California) |
| **Primary ML Models** | XGBoost (Asthma), XGBoost (Cardiovascular), Neural Network (Toxic Release) |
| **Core Composite Metric** | Health Vulnerability Index (HVI) |
| **Backend API** | FastAPI (Uvicorn server) deployed on Render |
| **Data Sources** | CalEnviroScreen 4.0, EPA ECHO, PurpleAir API, OpenStreetMap, Melissa Geocoding |

---

## 💡 Health Vulnerability Index (HVI) Breakdown

To provide a single comprehensive risk metric, we engineered the **Health Vulnerability Index (HVI)**, combining normalized weights across three key domains:

1. **Pollution Burden:** Air quality metrics (PM2.5, Ozone), diesel particulate matter, traffic density, and toxic chemical releases.
2. **Health Outcomes:** Pre-existing incidence rates for asthma emergency department visits, cardiovascular disease, and low birth weight.
3. **Socioeconomic Factors:** Poverty rate, educational attainment, unemployment rate, linguistic isolation, and housing burden.

---

## 🎯 Social & Environmental Impact

* **Environmental Justice Disparities:** Highlights severe risk overlaps in low-income industrial corridors, bringing visibility to overburdened communities.
* **Targeted Health Interventions:** Enables municipal health departments to identify the top 10 most vulnerable ZIP codes for priority asthma intervention programs.
* **Empowering Residents:** Provides individuals with localized pollution profiles and personalized, practical recommendations for reducing indoor air pollution exposure.

---

## 🛠 Actual Technical Implementation & Code Highlights (From GitHub)

### 1. Composite Health Vulnerability Index (HVI) Calculation (`health_index_score.py`)
Below is the actual column definition and feature aggregation used in our Python backend to calculate normalized health index scores across census tracts:

```python
import pandas as pd
from pathlib import Path
from sklearn.preprocessing import MinMaxScaler

# Feature domain columns from CalEnviroScreen 4.0
HEALTH_COLS = ["Asthma Pctl", "Cardiovascular Disease Pctl", "Low Birth Weight Pctl"]
ENV_COLS    = ["Pollution Burden Score"]
SDOH_COLS   = ["Education Pctl", "Poverty Pctl", "Linguistic Isolation Pctl", "Unemployment Pctl", "Housing Burden Pctl"]

ALL_COLS = HEALTH_COLS + ENV_COLS + SDOH_COLS

def compute_hvi_for_zips(df):
    """Normalizes health, environmental, and SDOH percentile features to produce HVI score."""
    scaler = MinMaxScaler()
    df[ALL_COLS] = scaler.fit_transform(df[ALL_COLS])
    
    # Weighted domain aggregation
    df['health_subscore'] = df[HEALTH_COLS].mean(axis=1)
    df['env_subscore']    = df[ENV_COLS].mean(axis=1)
    df['sdoh_subscore']   = df[SDOH_COLS].mean(axis=1)
    
    df['HVI_Score'] = (0.4 * df['health_subscore']) + (0.3 * df['env_subscore']) + (0.3 * df['sdoh_subscore'])
    return df
```

### 2. Multi-Model Inference & Percentile Rankings (`backend/main.py`)
Our production FastAPI backend loads both pre-trained XGBoost models (`best_xgb_model.pkl` & `cardiovascular_model.pkl`) to calculate real-time asthma and cardiovascular risk predictions along with state/county percentile rankings:

```python
from fastapi import FastAPI
from pathlib import Path
import pandas as pd
import joblib

app = FastAPI(title="Healthy Home API", version="1.0")

BASE_DIR = Path(__file__).resolve().parent
ASTHMA_MODEL_PATH = BASE_DIR / "models" / "best_xgb_model.pkl"
CARDIO_MODEL_PATH = BASE_DIR / "models" / "cardiovascular_model.pkl"

CARDIO_FEATURES = [
    "Ozone", "PM2.5", "Diesel PM", "Drinking Water", "Lead", "Pesticides",
    "Traffic", "Cleanup Sites", "Groundwater Threats", "Haz. Waste",
    "Imp. Water Bodies", "Solid Waste", "Education", "Linguistic Isolation",
    "Poverty", "Unemployment", "Housing Burden"
]

# Load ML pipelines
asthma_model = joblib.load(ASTHMA_MODEL_PATH)
cardio_model = joblib.load(CARDIO_MODEL_PATH)

# Predict across full feature matrix
asthma_df["pred_asthma"] = asthma_model.predict(asthma_X)
cardio_X = asthma_df[CARDIO_FEATURES]
asthma_df["pred_cardio"] = cardio_model.predict(cardio_X)
```

### 3. Geocoding & Melissa Table Lookup for Unknown ZIPs (`generate_predicted_zips.py`)
To ensure complete coverage across Southern California, we used the Melissa ZIP database (`Melissa_zipcodes.csv`) to map lat/lng coordinates and impute features for unseen ZIP codes:

```python
# Load Melissa geocode table (lat/lng for every US ZIP code)
melissa_df = pd.read_csv(BASE_DIR / "data" / "Melissa_zipcodes.csv")
melissa_df["ZipCode"] = melissa_df["ZipCode"].astype(str).str.zfill(5)
melissa_df = melissa_df.drop_duplicates(subset="ZipCode")

# Filter out ZIPs already indexed in the dataset
unindexed_zips = melissa_df[~melissa_df["ZipCode"].isin(indexed_zips)]
```
