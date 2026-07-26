---
title: "Predicting Marketing Success with Machine Learning"
description: "Customer targeting model built on 40,000+ UCI bank marketing records achieving 85% accuracy using Random Forest and Logistic Regression."
pubDate: "May 12 2026"
heroImage: "/marketing-ml.webp"
badge: "Machine Learning"
tags: ["Python", "scikit-learn", "SQL", "Class Imbalance", "Random Forest"]
---

<!-- 📊 KPI Summary Grid -->
<div class="grid grid-cols-1 sm:grid-cols-3 gap-4 my-6">
  <div class="p-4 bg-base-200/80 rounded-xl border border-base-300 shadow-sm flex flex-col justify-between">
    <span class="text-xs font-semibold text-base-content/70 uppercase tracking-wider">Model Accuracy</span>
    <div class="text-2xl md:text-3xl font-extrabold text-primary my-1">85%</div>
    <p class="text-xs text-base-content/80 mt-1">Random Forest Classifier</p>
  </div>
  <div class="p-4 bg-base-200/80 rounded-xl border border-base-300 shadow-sm flex flex-col justify-between">
    <span class="text-xs font-semibold text-base-content/70 uppercase tracking-wider">Dataset Scale</span>
    <div class="text-2xl md:text-3xl font-extrabold text-primary my-1">40,000+ Rows</div>
    <p class="text-xs text-base-content/80 mt-1">UCI Bank Marketing Records</p>
  </div>
  <div class="p-4 bg-base-200/80 rounded-xl border border-base-300 shadow-sm flex flex-col justify-between">
    <span class="text-xs font-semibold text-base-content/70 uppercase tracking-wider">Class Handling</span>
    <div class="text-2xl md:text-3xl font-extrabold text-primary my-1">Weighted Recall</div>
    <p class="text-xs text-base-content/80 mt-1">Class Imbalance & Threshold Tuning</p>
  </div>
</div>

## Executive Summary

Direct marketing campaigns incur significant operational costs for financial institutions. This project predicts whether a customer will subscribe to a term deposit using the **UCI Bank Marketing Dataset** (40,000+ customer records).

By combining **Logistic Regression** (for statistical interpretability) and a **Random Forest Classifier** (for non-linear predictive accuracy), this hybrid machine learning model identifies high-probability subscribers while preventing wasted outreach efforts.

---

## 🖥 Visual Model Overview

The model evaluates customer demographics, past campaign contacts, and macroeconomic indicators (Euribor 3-month rate, CPI, Consumer Confidence Index) to rank customer conversion probability.

![Bank Marketing Machine Learning](/marketing-ml.webp)

* 💻 **GitHub Repository:** [AaronVillegas5/uci-bank-marketing-analysis](https://github.com/AaronVillegas5/uci-bank-marketing-analysis)

---

## 📊 Model Performance Comparison

| Metric / Model | Baseline Logistic Regression | Tuned Random Forest |
| :--- | :--- | :--- |
| **Accuracy** | 83.2% | **85.1%** |
| **Recall (Subscriber Class)** | 0.54 | **0.62** |
| **ROC-AUC Score** | 0.78 | **0.84** |
| **Feature Leakage Handling** | Dropped `duration` feature | Dropped `duration` feature |
| **Primary Predictor** | Euribor 3-Month Rate | Euribor & Age |

---

## 💡 Key Business Insights

* **Macroeconomic Sensitivity:** The Euribor 3-month interest rate and Consumer Price Index were the strongest overall predictors of customer term deposit subscriptions.
* **Feature Leakage Prevention:** Call `duration` is highly correlated with subscription success but is unknown *before* a call is placed. Dropping `duration` ensured realistic pre-call predictive utility.
* **Handling Class Imbalance:** Term deposit conversions represent ~12% of baseline contacts. Applying `class_weight='balanced'` and custom decision thresholds significantly improved positive class recall without sacrificing precision.

---

## 🛠 Interactive Code Highlights (From GitHub)

<div class="my-6 rounded-xl border border-base-300 bg-base-200/40 p-4 shadow-sm">
  <div role="tablist" class="tabs tabs-lifted">
    <!-- Tab 1 -->
    <input type="radio" name="code_tabs_bank" role="tab" class="tab font-semibold" aria-label="Feature Preprocessing" checked />
    <div role="tabpanel" class="tab-content bg-base-100 border-base-300 rounded-box p-4 overflow-x-auto max-w-full">
      <p class="text-xs text-base-content/80 mb-3"><b>Python / Pandas:</b> Preprocessing features and resolving data leakage:</p>

```python
import pandas as pd
from sklearn.model_selection import train_test_split

# Load dataset and drop call duration to prevent feature leakage
df = pd.read_csv('bank-additional-full.csv', sep=';')
df_clean = df.drop(columns=['duration'])

# Engineer feature for previous contact history
df_clean['never_contacted'] = (df_clean['pdays'] == 999).astype(int)

# One-hot encoding for categorical variables
df_encoded = pd.get_dummies(df_clean, drop_first=True)
X = df_encoded.drop(columns=['y_yes'])
y = df_encoded['y_yes']

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)
```

    </div>

    <!-- Tab 2 -->
    <input type="radio" name="code_tabs_bank" role="tab" class="tab font-semibold" aria-label="Random Forest Pipeline" />
    <div role="tabpanel" class="tab-content bg-base-100 border-base-300 rounded-box p-4 overflow-x-auto max-w-full">
      <p class="text-xs text-base-content/80 mb-3"><b>scikit-learn:</b> Training balanced Random Forest classifier with feature importance extraction:</p>

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report, roc_auc_score

# Initialize Random Forest with balanced class weighting
rf_model = RandomForestClassifier(
    n_estimators=200,
    max_depth=12,
    class_weight='balanced',
    random_state=42
)

rf_model.fit(X_train, y_train)
y_pred = rf_model.predict(X_test)
y_proba = rf_model.predict_proba(X_test)[:, 1]

print("ROC-AUC Score:", roc_auc_score(y_test, y_proba))
print(classification_report(y_test, y_pred))
```

    </div>

    <!-- Tab 3 -->
    <input type="radio" name="code_tabs_bank" role="tab" class="tab font-semibold" aria-label="Feature Importance" />
    <div role="tabpanel" class="tab-content bg-base-100 border-base-300 rounded-box p-4 overflow-x-auto max-w-full">
      <p class="text-xs text-base-content/80 mb-3"><b>Feature Importance:</b> Extracting top predictive economic & demographic drivers:</p>

```python
import numpy as np

# Extract top feature importances
importances = rf_model.feature_importances_
feature_names = X.columns
feature_importance_df = pd.DataFrame({
    'Feature': feature_names,
    'Importance': importances
}).sort_values(by='Importance', ascending=False)

print("Top 5 Predictive Features:")
print(feature_importance_df.head(5))
```

    </div>
  </div>
</div>
