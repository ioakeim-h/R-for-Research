# Outliers

An outlier is a data point that differs significantly from the rest, skewing the mean and inflating the standard deviation. This means that outliers contribute to biased results, especially for small sample sizes. Removing outliers creates a more representative sample of the majority, but risks losing important insights. Before doing so, we can consider the following:

**Did they do the study correctly?**

-   If no (e.g. measurement errors, data corruption): Outliers may be removable, but first assess whether they exposed flaws in study design (e.g., biased sampling).

-   If yes: Outliers could signal uncontrolled variables (e.g., an unrecorded experimental condition).

**Do they belong to the target population?**

-   If no: Removal may be justified. However, "target population" definitions can be arbitrary - what seems like an "out-of-population" outlier might actually be a valid edge case revealing uncontrolled variables (e.g., a rare medical condition in a health study).

-   If yes: Outliers reflect real variation and may indicate meaningful trends.

## Univariate outlier detection

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

## Multivariate outlier detection

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
