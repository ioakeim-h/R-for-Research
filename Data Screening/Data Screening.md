## Normality

The term refers to the assumption that the sampling distribution is normal. The sampling distribution is not something we have access to. It is the population distribution that we look at, which according to the Central Limit Theorem, if the sample data are approximately normal then the sampling distribution will also be.

-   Sample distribution: Data distribution from one specific sample.

-   Sampling distribution: Distribution of a statistic (e.g., mean) calculated from many samples of the same size.

In big samples where N \> 30, normality can be assumed regardless of the shape of the data. If N \< 30 and the normality assumption is not met, we may consider using non-parametric tests.

We can look for normality visually, look at values that quantify aspects of a distribution (i.e. skew and kurtosis) and compare the distribution we have to a normal distribution to see if it is different.

### Visualizing Normality

#### Density Histograms

-   **Definition**: A density histogram displays the probability density of the data rather than the raw counts, which are shown in a frequency histogram.

-   **Y-Axis**: The y-axis represents the density, which is calculated as the frequency of observations in each bin divided by the total number of observations and the width of the bin. This ensures that the area under the histogram sums to 1.

-   **Use Case**: It is useful for comparing distributions between different datasets or variables, especially when they have different sample sizes. Density histograms allow for a more normalized view of the data distribution.

```R
ggplot(myData, aes(x = numerica_var)) + 
  geom_histogram(aes(y = after_stat(density)), binwidth = 2) + 
  labs(x = "xLabel", y = "Density") +
  stat_function(
    fun = dnorm, 
    args = list(mean(myData$numerica_var, na.rm = T), sd = sd(myData$numerica_var, na.rm = T)),
    colour = "black",
    linewidth = 0.5
  )
```

The `stat_function()` produces a normal curve using the function `dnorm()`. The rest of the command specifies the mean as being the mean of the numerical_var after removing any missing values and the standard deviation as being that of numerical_var.

#### Quantile-Quantile (QQ) Plots

A quantile is the proportion of cases we find below a certain value. A QQ plot displays the cumulative values we have in our data against the cumulative probability of a particular distribution (in this case we specify a normal distribution). Each value is compared to the expected value that the score should have in a normal distribution. If the data are normally distributed then the actual scores will have the same distribution as the score expected from a normal distribution, and you will get a straight diagonal line. Deviations from the line show deviations from normality.

```R 
ggplot(myData, aes(sample = numerical_var)) + 
  stat_qq() + 
  stat_qq_line() +
  labs(x = "Theoretical quantiles", y = "Sample quantiles")
```

-   **Y-axis:** Your actual (sample) data quantiles
-   **X-axis:** Theoretical quantiles from a normal distribution

### Quantifying Normality

**Skewness and Kurtosis**

Plots can be misinterpreted, so we can further explore the distribution of variables with numbers. We are interested in getting the skewness and kurtosis of each variable we want to explore. We can get these statistics using the describe() function from the `psych` package.

The `describe()` function returns a variety of statistics, but we can select only n (sample size [for contrasting with Central Limit Theorem]), skewness and kurtosis.

```R    
# Returns skewness and kurtosis for all variables
describe(myData)[c("n",skew","kurtosis")]

# Returns skewness and kurtosis for the selected variables
describe(myData[,c("variable1", "variable2")])[c("n",skew","kurtosis")]
```

The values of skewness and kurtosis should be zero in a normal distribution. Positive values of skew indicate a pile-up of scores on the left of the distribution, whereas negative values indicate a pile-up on the right. Positive values of kurtosis indicate a pointy and heavy-tailed distribution, whereas negative values indicate a flat and light-tailed distribution. The further the value is from zero, the more likely it is that the data are not normally distributed.

### Testing for Normality

Another way to explore the distribution of the data is with normality tests. These tests compare the sample distribution to a normal distribution with the same mean and standard deviation, but differ in their methodologies and sensitivities:

1.  **Shapiro-Wilk test**

-   Compares the sample data to a normal distribution.
-   Method: Based on the correlation between the sample quantiles and the expected quantiles of a normal distribution (Q-Q plot concept).
-   Sensitivity: Best for small to moderate sample sizes (n \< 50).

```R         
# Shapiro-Wilk
shapiro.test(myData$numerical_var)
```

2.  **Anderson-Darling test**

-   Compares the empirical distribution function (EDF) of the sample to the expected normal distribution.
-   Method: A modified version of the Kolmogorov-Smirnov test that gives more weight to the tails.
-   Sensitivity: Robust for small and large samples. Better at detecting deviations in the tails of the distribution.

```R         
# Anderson-Darling (from nortest package)
nortest::ad.test(myData$numerical_var)
```

The Kolmogorov-Smirnov (KS) test is also an option, but it is less powerful than specialized tests like Shapiro-Wilk (SW) or Anderson-Darling (AD). This is because it is designed to detect deviations from any distribution, not just normality, and is relatively insensitive to tail behavior. The standard KS test assumes you know the true μ and σ of the normal distribution, which is unrealistic in practice. Although this can be adjusted with the Lilliefors correction, KS would still be inclined towards a higher type II error (missing true non-normality) compared to SW or AD.

Tests of normality have their limitations and their results are not absolute. Always **pair with visual checks** and **skew and kurtosis values**.

### Normality Across Levels

A deeper understanding of the distribution of our data can be obtained by studying the distribution of each level within a factor. When factor variables are present in the data, we can assess normality within each level to capture group-specific patterns.

**A plot for each level**

After filtering the data on a particular level, we can plot the distribution using any appropriate graph, such as a histogram or a qqplot.

```R         
myDataSubset <- subset(myData, myData$factor_variable == "myLevel")
# code for the plot goes here
```

**Descriptive statistics for each level**

The `by()` function applies a function to each level of a factor and groups the results by that level. Using it along with the `describe()` function, it returns the descriptive statistics for all (or the selected) variables, split by the levels in the factor variable.

```R         
# Return descriptive stats for all variables, split by a factor
by(myData, myData$factor_variable, FUN = describe)

# Return descriptive stats for the selected variables, split by a factor
by(myData[, c("var1","var2")], myData$factor_variable, FUN = describe)
```

**Normality tests for each level**

Similarly, the `by()` function can be used with the Shapiro-Wilk test or the Anderson-Darling test to explore the distribution of data across the levels of a factor.

```R         
# Extract only p-values: Shapiro-Wilk
by(myData, myData$factor_variable, function(x) shapiro.test(x$numerical_var)$p.value)

# Anderson-Darling
by(myData, myData$factor_variable, function(x) nortest::ad.test(x$numerical_var)$p.value)
```

## Multicollinearity 

**Note (to revisit in Regression Chapter of Discovering Statistics Using R textbook, p. 292):** Multicollinearity is only relevant when you have multiple predictors (e.g., in multiple regression or factorial ANOVA). It's not a concern in simple t-tests or one-way ANOVA with one factor.

Multicollinearity undermines the stability and interpretability of model coefficients. Always check for valid predictors first—before testing assumptions like homoscedasticity or homogeneity of variance.

## Homogeneity and Homoscedasticity

Homogeneity of variance (or homoscedasticity) means that the variances (squared standard deviations) of the scores in different groups or across different conditions are approximately equal. Put in another way, the assumption asserts that as we go through levels of one variable, the variance of the other should not change. The two terms mean essentially the same thing — equal variances — but are used in different contexts:

-   **Homogeneity of variance**: used when comparing groups (e.g. ANOVA). If we have collected groups of data (i.e. categorical data) then this means that the variance of the outcome variable or variables should be the same in each of these groups.

-   **Homoscedasticity**: used in regression or continuous predictors describing equal variance of residuals. If we have collected continuous data (such as in correlational designs), this assumption means that the variance of one variable should be stable at all levels of the other variable.

### Testing homogeneity of variance

While visual methods like boxplots are commonly used to check homogeneity of variance, they do have limitations in formal research - which is why statistical tests are prioritized in publications. The difference between using visual methods for homogeneity of variance and homoscedasticity, is that the former explores variance across categorical groups while the latter does so across continues values. This makes visual methods for testing homogeneity of variance highly subjective, as there is no objective threshold for "equal enough" variance. Having said that, they can still be used to aid in the identification of problematic groups.

**Levene's Test**

For groups of data we tend to use Levene's test to assess homogeneity of variance. It tests the null hypothesis that the variances in different groups are equal (i.e. the difference between the variances is zero). An important **limitation** of this test is that when the sample size is large, small differences in group variances can produce a significant result.

-   If Levene's test is significant at *p* \< .05, the assumption of homogeneity of variance has been violated.
-   If it is not significant (*p* \> .05), then the variances are roughly equal and the assumption is tenable.

We can test for homogeneity of variance using the `leveneTest()` function from the `car` package. The function takes two variables: first the outcome variable of which we want to test the variances; and second, the factor variable. Levene's test will by default center the variables using the median (which is slightly preferable) but the mean can be used as well.

```R         
leveneTest(myData$outcome_var, myData$factor_var, center = median)
```

The output highlights significance by the `Pr (>F)` value.

**Fligner-Killeen Test**

The Fligner-Killeen test is a non-parametric method for testing homogeneity of variances across groups, and it is known for its robustness to departures from normality. It performs well with large sample sizes and can be used in addition to Levene's test.

-   If *p* \> .05, the assumption for homogeneity of variance is met.
-   If *p* \< .05, variances differ across groups and the assumption is violated.

The Fligner-Killeen test is in base R and can be used with the `fligner.test()` function:

```R         
fligner.test(outcome_var ~ factor_var, data = myData)
```

### Testing homoscedasticity 

**Note (to revisit in Regression Chapter of Discovering Statistics Using R textbook, p. 294):** In correlational analyses like regression, homoscedasticity is typically assessed using graphical methods.


## Outliers

An outlier is a data point that differs significantly from the rest, skewing the mean and inflating the standard deviation. This means that outliers contribute to biased results, especially for small sample sizes. Removing outliers creates a more representative sample of the majority, but risks losing important insights. Before doing so, we can consider the following:

**Did they do the study correctly?**

-   If no (e.g. measurement errors, data corruption): Outliers may be removable, but first assess whether they exposed flaws in design (e.g., biased sampling).

-   If yes: Outliers could signal uncontrolled variables (e.g., an unrecorded experimental condition).

**Do they belong to the target population?**

-   If no: Removal may be justified. However, "target population" definitions can be arbitrary - what seems like an "out-of-population" outlier might actually be a valid edge case revealing uncontrolled variables (e.g., a rare medical condition in a health study).

-   If yes: Outliers reflect real variation and may indicate meaningful trends.

### Univariate outlier detection

Univariate outliers are extreme vales in a single variable. Univariate outlier detection looks at each variable separately, identifying extreme values within that single variable’s distribution.

The Interquartile Range (IQR) method is used to detect outliers in a dataset by focusing on the central 50% of the data. It works by calculating the first quartile (Q1) and third quartile (Q3), then computing the IQR as the difference between them (IQR = Q3 - Q1). Outliers are defined as values that fall below Q1 − 1.5×IQR or above Q3 + 1.5×IQR. This method is considered robust because it relies on the median and quartiles rather than the mean, making it less sensitive to extreme values. As a result, it performs well in most scenarios where data may not follow a normal distribution or where outliers could skew results.

A more conservative approach to outlier detection involves using the 3 × IQR instead of 1.5 × IQR. This sets wider boundaries so only more extreme values are flagged as outliers. It is useful when the data naturally has high variability or when you want to reduce the risk of falsely labeling valid data points as outliers. It's often preferred in large datasets, heavy-tailed distributions, or when preserving more data is important.

-   1.5 x IQR: mild outlier detection
-   3 x IQR: extreme outlier detection

In R, we can create a function for detecting outliers using the IQR method. This will allow us to apply the function to a single variable or to all. As shown below, the function takes by default a threshold of 1.5, but if required, this can be adjusted to 3 upon calling the function.

```R         
detect_outliers_iqr <- function(data, threshold = 1.5) {
  # Calculate quartiles and IQR
  Q1 <- quantile(data, 0.25, na.rm = TRUE)
  Q3 <- quantile(data, 0.75, na.rm = TRUE)
  IQR <- Q3 - Q1
  
  # Define lower and upper bounds for outliers
  lower <- Q1 - threshold * IQR
  upper <- Q3 + threshold * IQR
  
  # Return a logical vector indicating outliers
  data < lower | data > upper
}

# Apply to a single variable 
outliers_vector <- detect_outliers_iqr(myData$continuous_var)
View(myData[outliers_vector, ])

# Apply to all variables 
outliers_df <- sapply(myData, detect_outliers_iqr) # Select columns if needed
rows_with_outliers <- apply(outliers_df, 1, any)
View(myData[rows_with_outliers, ])
```

To make better decisions about handling outliers, we can visualize them with boxplots. The base R `boxplot()` function provides a quick and easy way to do this but lacks publication-quality aesthetics. For publication-ready plots, `ggplot2` offers more customization and polished visuals, though it is much more complex depending on the data.

```R         
# Plot outliers in a single variable: range specifies threshold
boxplot(myData$continuous_var, range = 3)

# Plot outliers in a single variable split on group
boxplot(continuous_var ~ group, data = myData, range = 3)
```

The IQR method is well-established for outlier detection, but ignores relationships between variables. When multiple variables are present, it may be best to look into multivariate outlier detection techniques.

### Multivariate outlier detection

Multivariate outliers are unusual combinations of values across multiple variables within one observation (row). Multivariate outlier detection considers the combination of values across multiple variables simultaneously, identifying points that are unusual in the joint distribution, even if individual values seem normal alone.

One way to detect multivariate outliers is by calculating **Mahalanobis distance** - a measure of the distance between a point and the center (mean) of a distribution, relative to the spread (variance) and correlation structure (covariance). It is a flexible method as it adjusts for variable scales (e.g., when one variable is in meters and another in kilograms).

Mahalanobis distances follow a chi-square distribution with degrees of freedom equal to the number of *variables being tested* (not all available variables), excluding categorical variables. For the identification of outliers, a chi-square threshold can be used to decide if a distance is extreme. Any distance above the threshold is an outlier.

**Robust Mahalanobis distance**

The Mahalanobis distance requires the parameters of the distribution: mean and covariance matrix. These are unknown and must be estimated from the data. A problem arises here because anomalies in the data can distort the parameter estimates, with the effect of making these points appear less anomalous than they really are. Minimum Covariance Determinant (MCD) is a method for estimating the mean and covariance matrix in a way that tries to minimize the influence of anomalies. The idea is to estimate these parameters from a subset of the data that has been chosen to (hopefully) not contain anomalies. MCD selects the subset of the data that is most tightly distributed. This is to exclude anomalies, which are likely to lie further away from the rest of the data.

In R, the procedure entails:

```R         
library(MASS)       # For robust covariance estimation
library(ggplot2)

# Step 0: Filter out irrelevant (i.e. untested) and factor variables
data <- subset(myData, select = -c(var_to_exclude))

# Step 1: Calculate robust center (median) and covariance using Minimum Covariance Determinant (MCD)
cov_obj <- cov.rob(data, method = "mcd") 

# Step 2: Extract robust center (median vector) and covariance matrix
center <- cov_obj$center
cov_matrix <- cov_obj$cov

# Step 3: Calculate Mahalanobis distances using robust center and covariance
mahalanobis_dist <- mahalanobis(data, center, cov_matrix)

# Step 4: Determine threshold at 97.5th percentile of Chi-square (95% confidence for outliers)
threshold <- qchisq(0.975, df = ncol(data)) 
outlier_indices <- which(mahalanobis_dist > threshold)

# Step 5: Review outliers within the original dataset
View(myData[outlier_indices, ])

# Step 6: Consider whether to remove
clean_data <- myData[-outliers, ]
```

Multivariate outliers are difficult to visualize, but their interpretation can be combined with univariate box plots.



