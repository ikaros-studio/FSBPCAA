# Documentative Overview of Stata Commands Used in the Thesis

This document provides a comprehensive description of the Stata commands that were used throughout the thesis analyses. The commands described below were applied to perform reliability analysis, descriptive statistics, correlation analysis, regression (including interactions), ANOVA with post-hoc comparisons, paired and nonparametric tests, data cleaning, exporting results, and graphing. Each section explains the purpose and implementation of the commands as they were used in the project.

---

## 1. Reliability Analysis (Cronbach’s Alpha)

To assess the internal consistency of multi-item scales, such as the climate change belief scale, the `alpha` function was employed.

```stata
alpha Q1 Q2 Q3 Q4, asis
```

- **Purpose:** This command computed Cronbach’s alpha for items Q1 through Q4 to assess the reliability of the scale.
- **Details:** The `asis` option ensured that the items were used in their original form without standardization.

---

## 2. Descriptive Statistics

Descriptive statistics were generated to summarize the key variables in the dataset.

```stata
summarize var1 var2 var3, detail
```

- **Purpose:** This command provided summary statistics, including the mean, standard deviation, minimum, maximum, and detailed percentiles for each variable.

---

## 3. Correlation Analysis

To examine the relationships among key variables, a Pearson correlation matrix was computed.

```stata
correlate var1 var2 var3, covariance
```

- **Purpose:** The command calculated Pearson correlation coefficients along with the covariance matrix to explore associations between variables.

---

## 4. Regression Analysis

### 4.1 Multiple Linear Regression

Multiple regression analysis was conducted to examine the effects of predictor variables on an outcome variable while controlling for covariates.

```stata
regress belief_post belief_pre age gender education location
```

- **Purpose:** This regression model predicted the post-intervention variable (`belief_post`) based on its baseline measure (`belief_pre`) and several control variables.

### 4.2 Including Interaction Terms

To investigate interaction effects between two continuous variables, the following command was executed:

```stata
regress outcome c.var1##c.var2
```

- **Purpose:** The command incorporated both main effects and the interaction effect into the regression model.
- **Details:** The `c.` prefix indicated that both variables were continuous, while the `##` operator ensured the inclusion of main effects and interaction terms.

---

## 5. ANOVA and Post-Hoc Comparisons

### One-Way ANOVA

To compare means across different groups, a one-way ANOVA was performed.

```stata
oneway outcome group, tabulate
```

- **Purpose:** This analysis examined whether significant differences existed between groups for the dependent variable `outcome`.

### Pairwise Comparisons (Tukey’s HSD)

After conducting ANOVA, pairwise comparisons were carried out using Tukey’s HSD test.

```stata
pwmean outcome, over(group) mcompare(tukey)
```

- **Purpose:** The test adjusted for multiple comparisons to prevent inflated Type I error.

---

## 6. Paired and Nonparametric Tests

### 6.1 Paired t-test

For comparing pre- and post-intervention scores when normality assumptions were met, the `ttest` function was used.

```stata
ttest var_pre == var_post
```

- **Purpose:** This command tested whether the mean difference between paired samples was statistically significant.

### 6.2 Wilcoxon Signed-Rank Test

When the normality assumption was not met, the Wilcoxon signed-rank test was performed as a nonparametric alternative.

```stata
signrank var_pre = var_post
```

- **Purpose:** The test assessed whether the median difference between pre- and post-intervention scores was significantly different from zero.

---

## 7. Data Preprocessing and Cleaning

### Renaming Variables

To rename a variable for clarity:

```stata
rename oldname newname
```

### Generating New Variables

To create a new variable representing the product of two existing variables:

```stata
generate newvar = var1 * var2
```

### Handling Missing Data

To summarize the extent of missing data in the dataset:

```stata
misstable summarize var1 var2 var3
```

- **Purpose:** This command identified missing values across key variables, allowing for appropriate handling strategies.

---

## 8. Exporting Results

### Exporting Regression Output

To export regression results for inclusion in reports, the `outreg2` command was used:

```stata
outreg2 using "results.doc", word replace
```

Alternatively, the `esttab` command was utilized:

```stata
esttab using results.xls, replace
```

- **Purpose:** These commands formatted and exported statistical output in Word and Excel formats.

---

## 9. Graphing

### 9.1 Scatter Plot with a Fit Line

To visualize the relationship between two variables, a scatter plot was generated:

```stata
scatter yvar xvar, yline(, lpattern(dash))
```

- **Purpose:** This graph provided a visual representation of the correlation between `xvar` and `yvar`.

### 9.2 Histogram with a Normal Distribution

To examine the distribution of a variable with an overlaid normal curve:

```stata
histogram var1, normal
```

### 9.3 Box Plot

To compare distributions across groups:

```stata
graph box yvar, over(group)
```

- **Purpose:** This box plot facilitated visual comparisons of `yvar` across different categories of `group`.

---

## 10. Logistic Regression

To analyze binary outcome variables, logistic regression was performed:

```stata
logit binary_outcome var1 var2 var3
```

- **Purpose:** This model estimated the probability of `binary_outcome` as a function of predictor variables.

---

This document provides a structured overview of the Stata commands applied in the thesis. Each command played a crucial role in statistical analysis and result reporting.
