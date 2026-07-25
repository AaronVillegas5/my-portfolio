---
title: "Walmart Sales & Demand Volatility Analysis"
description: "End-to-end Python, SQL, and Tableau analytics pipeline evaluating store-level demand volatility and holiday sales lifts for inventory planning."
pubDate: "Apr 15 2026"
heroImage: "/walmart-analytics.webp"
badge: "Data Analytics"
tags: ["Python", "SQL", "Tableau", "Pandas", "Retail Analytics"]
---

## Executive Summary
This project analyzes two years of weekly sales data across 45 Walmart stores to identify key drivers of revenue, seasonal demand spikes, and store-level volatility. By evaluating the relationship between historical sales and external economic indicators (CPI, fuel prices, unemployment, and temperature), this analysis provides actionable, data-driven recommendations to optimize inventory planning, staffing allocation, and localized sales forecasting.

---

## 🖥 Interactive Tableau Dashboard

I designed a comprehensive Tableau dashboard tracking core business KPIs (YoY growth, demand variability, and store rankings) to directly support executive inventory planning and staffing decisions.

![Walmart Sales Dashboard](/walmart-analytics.webp)

[**Click here to view the interactive Tableau Dashboard**](https://public.tableau.com/app/profile/aaron.villegas3123/viz/WalmartSalesAnalysis_17802549747750/KPIDashboard)

---

## 📊 Key Performance Indicators (KPIs)

| Metric | Value |
| :--- | :--- |
| **Total Sales Volume** | $6,737,218,987 |
| **Average Weekly Sales (All Stores)** | $47,113,419 |
| **Average Weekly Sales (Per Store)** | $1,046,965 |
| **Broader Holiday Sales Lift** | +7.84% |
| **Christmas Week Sales Lift** | +69.23% |
| **Top 5 Stores Revenue Contribution** | 22.5% |

---

## 💡 Business Insights & Supply Chain Impact

* **Seasonal Demand Spikes:** Holiday weeks drive a baseline 7.84% increase in sales, but Christmas week acts as a massive outlier, generating a **69% revenue lift** compared to non-holiday weeks. 
* **Revenue Concentration:** Sales are heavily skewed, with the top 5 performing stores (Stores 20, 4, 14, 13, and 2) generating over **22% of total revenue**.
* **Store-Level Volatility (Risk Assessment):** Variance is an important factor in forecasting sales. Standard deviation is misleading for stores with different baselines, so I measured risk using the **Coefficient of Variation (CV)**. Stores 35, 7, 15, 29, and 23 exhibited the highest relative volatility, representing higher risk for overstock or stockouts.
* **Local Economic Sensitivity:** Across the entire network, external factors (Temp, Fuel, CPI, Unemployment) showed near-zero linear correlation with sales. However, isolating the data by store revealed that *specific* locations are highly sensitive to these external factors, indicating that broad national models are insufficient for accurate local forecasting.

## 🎯 Actionable Recommendations

1. **Dynamic Inventory Allocation:** Transition highly volatile stores (highest CV/variance) to a more flexible inventory strategy to lower risks of low stock, while keeping fixed inventory models for the most consistent stores (Stores 31, 44, 43, 30, and 37).
2. **Localized Forecasting Models:** Shift away from national macroeconomic forecasting. Incorporate store-specific weather and economic data into localized machine learning models to improve week-over-week demand prediction accuracy.
3. **Strategic Holiday Staffing:** Scale staffing and logistics significantly during the week of December 19-25, which independently drives the massive 69% seasonal lift.

---

## 🛠 Technical Methodology & Code Highlights

### Data Pipeline & Cleaning
Queried a PostgreSQL database containing ~6,400 weekly sales records using custom Python ETL loader scripts. Data integrity was verified by checking for nulls and duplicate entries across store/date primary keys.

### Measuring Relative Volatility (CV) via SQL
To accurately identify which stores carried the highest inventory risk, I calculated the Coefficient of Variation (Standard Deviation / Mean) directly in PostgreSQL:

```sql
SELECT 
    store,
    STDDEV(weekly_sales) / AVG(weekly_sales) AS sales_cv
FROM sales
GROUP BY store
ORDER BY sales_cv DESC;
```
### Isolating the Christmas Anomaly
To separate the general "Holiday" flag from the actual Christmas impact, I engineered specific Boolean flags in Pandas using datetime indexing:

```python
# Create specific flag for the week of Christmas
df['is_christmas_week'] = df['date'].apply(lambda x: x.month == 12 and 19 <= x.day <= 25)

grouped = df.groupby('is_christmas_week')['weekly_sales'].mean()
print(f"Percent increase in sales during Christmas week: {(grouped[True] - grouped[False]) / grouped[False] * 100:.2f}%")
```
### Year-Over-Year (YoY) Growth Tracking
Extracted year-over-year overlapping months to calculate growth trends accurately across all retail locations.
```sql
SELECT
    store,
    EXTRACT(YEAR FROM date) AS year,
    SUM(weekly_sales) AS total_sales
FROM sales
WHERE EXTRACT(MONTH FROM date) BETWEEN 2 AND 10
GROUP BY store, year
ORDER BY store, year;
```