---
title: "A/B Testing Analysis of Digital Marketing Campaigns"
description: "Statistical analysis of a 588K-user marketing dataset evaluating ad vs. PSA conversion rates using hypothesis testing and confidence intervals."
pubDate: "May 10 2026"
heroImage: "/ab-testing.webp"
badge: "A/B Testing"
tags: ["Python", "statsmodels", "A/B Testing", "Hypothesis Testing", "Pandas"]
---

## Executive Summary

Digital advertising decisions require rigorous statistical validation to distinguish genuine campaign lifts from random baseline variation. This project conducts a comprehensive end-to-end A/B test analysis on a **588,000-user marketing dataset**, evaluating whether showing an advertisement (treatment group) yields a statistically significant increase in user purchase conversions compared to a Public Service Announcement (control PSA group).

By implementing pre-test power analysis, a two-proportion z-test, and 95% confidence interval estimation, the analysis proves a **statistically significant 0.7% conversion lift** for the ad group ($p < 0.05$).

---

## 🖥 Experiment Summary & Visual Lift Metrics

The experiment compares ~564,000 treatment users (Ad) against ~24,000 control users (PSA) across metrics including total ads seen, peak exposure day, and peak exposure hour.

![A/B Testing Conversion Rate Comparison](/ab-testing.webp)

* 💻 **GitHub Repository:** [AaronVillegas5/ABTest-MarketingCampaign](https://github.com/AaronVillegas5/ABTest-MarketingCampaign)

---

## 📊 Key Performance Indicators (KPIs) & Experiment Summary

| Metric / Parameter | Value |
| :--- | :--- |
| **Total Sample Size** | 588,101 users |
| **Treatment Group (Ad)** | 564,577 users (~96%) |
| **Control Group (PSA)** | 23,524 users (~4%) |
| **Ad Group Conversion Rate** | 2.55% |
| **PSA Group Conversion Rate** | 1.79% |
| **Absolute Conversion Lift** | **+0.76%** |
| **Statistical Significance** | **$p < 0.001$** (Reject $H_0$) |
| **95% Confidence Interval** | $[0.0059, 0.0093]$ (Difference in proportions) |

---

## 💡 Key Business Insights

* **Statistically Significant Lifts:** The treatment group achieved a $2.55\%$ conversion rate compared to $1.79\%$ for the control group. A two-proportion z-test yielded a $p$-value well below $\alpha = 0.05$, confirming the lift is not due to sampling noise.
* **Massive Business Impact at Scale:** While an absolute lift of $+0.76\%$ may appear modest, applying this lift across $500,000+$ impression targets generates thousands of incremental conversions and substantial revenue gains.
* **Unequal Sample Ratio Validation:** Power analysis confirmed that despite the 96/4 group split, both sample sizes exceeded the minimum statistical threshold required for $80\%$ statistical power at $\alpha = 0.05$.

---

## 🛠 Technical Methodology & Code Highlights (From GitHub)

### 1. Sample Size Validation & Power Analysis
Before testing hypotheses, we verified statistical power given the unequal group ratio ($r = n_{PSA} / n_{ad} \approx 0.0417$):

```python
import statsmodels.stats.api as sms

# Pre-experiment power analysis for unequal sample sizes
effect_size = sms.proportion_effectsize(0.0255, 0.0179)
required_n_ad = sms.NormalIndPower().solve_power(
    effect_size=effect_size, 
    power=0.80, 
    alpha=0.05, 
    ratio=0.0417
)

print(f"Required Treatment Sample Size: {int(required_n_ad):,}")
# Actual sample size (564,577) > Required sample size -> Test is statistically valid
```

### 2. Two-Proportion Z-Test Execution
Using `statsmodels`, we tested the null hypothesis $H_0: p_{ad} = p_{PSA}$ against the two-sided alternative $H_1: p_{ad} \neq p_{PSA}$:

```python
from statsmodels.stats.proportion import proportions_ztest, proportion_confint
import numpy as np

# Aggregate conversion counts and total observations
count_converted = np.array([count_ad_converted, count_psa_converted])
n_obs = np.array([n_ad, n_psa])

# Execute Two-Proportion Z-Test
z_stat, p_value = proportions_ztest(count_converted, n_obs)

print(f"Z-statistic: {z_stat:.4f}")
print(f"p-value: {p_value:.4e}")

if p_value < 0.05:
    print("Decision: Reject H0 — Significant conversion lift observed.")
```

### 3. Difference Confidence Interval Calculation
Calculating the 95% confidence interval around the difference in conversion proportions ($\hat{p}_{ad} - \hat{p}_{PSA}$):

```python
import scipy.stats as stats

# Standard error of difference in proportions
se_diff = np.sqrt(
    (p_ad * (1 - p_ad) / n_ad) + (p_psa * (1 - p_psa) / n_psa)
)

diff = p_ad - p_psa
z_critical = stats.norm.ppf(0.975)  # 95% confidence
ci_lower = diff - z_critical * se_diff
ci_upper = diff + z_critical * se_diff

print(f"95% CI for Conversion Lift: [{ci_lower:.4f}, {ci_upper:.4f}]")
```
