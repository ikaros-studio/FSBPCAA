# Stata Commands Overview for Thesis Analyses

This document provides an overview of the common Stata commands used throughout the thesis. The analyses include reliability testing (Cronbach’s alpha), descriptive statistics, correlation analysis, regression (including interaction terms), ANOVA with post‐hoc comparisons, paired t-tests and nonparametric tests, data preprocessing, result exporting, and graphing.

---

## 1. Reliability Analysis (Cronbach’s Alpha)

To assess the internal consistency of multi-item scales (e.g., climate change belief, sustainable consumption), use the `alpha` command:

```stata
* Calculate Cronbach's alpha for a set of items (e.g., Q1, Q2, Q3, Q4)
alpha Q1 Q2 Q3 Q4, asis