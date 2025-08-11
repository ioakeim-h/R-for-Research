# Normality

Normality refers to the assumption that the sampling distribution is normal. However, the sampling distribution is not something we have access to. It is the population distribution that we look at, which according to the Central Limit Theorem, if the sample data are approximately normal, then the sampling distribution will also be.

-   Sample distribution: Data distribution from one specific sample.

-   Sampling distribution: Distribution of a statistic (e.g., mean) calculated from all possible samples of the same size.

In big samples where N \> 30, normality can be assumed regardless of the shape of the data. If N \< 30 and the normality assumption is not met, we may consider using non-parametric tests for our statistical analysis.

We can assess normality by visually examining the data, calculating measures such as skewness and kurtosis, and comparing the observed distribution to a normal distribution to determine any differences.

## Visualizing Normality

### Density Histograms

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

### Quantile-Quantile (QQ) Plots

A quantile is the proportion of cases we find below a certain value. A QQ plot displays the cumulative values we have in our data against the cumulative probability of a particular distribution (in this case we specify a normal distribution). Each value is compared to the expected value that the score should have in a normal distribution. If the data are normally distributed then the actual scores will have the same distribution as the score expected from a normal distribution, and you will get a straight diagonal line. Deviations from the line show deviations from normality.

```R 
ggplot(myData, aes(sample = numerical_var)) + 
  stat_qq() + 
  stat_qq_line() +
  labs(x = "Theoretical quantiles", y = "Sample quantiles")
```

-   **Y-axis:** Your actual (sample) data quantiles
-   **X-axis:** Theoretical quantiles from a normal distribution

## Quantifying Normality

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

## Testing for Normality

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

## Normality Across Levels

A deeper understanding of the distribution of our data can be obtained by studying the distribution of each level within a factor. When factor variables are present in the data, we can assess normality within each level to capture group-specific patterns.

**A plot for each level**

After filtering the data on a particular level, we can plot the distribution using any appropriate graph, such as a histogram or a qqplot.

```R         
myDataSubset <- subset(myData, myData$factor_variable == "myLevel")
# code for the plot goes here
```

**Descriptive statistics for each level (skewness and kurtosis)**

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
