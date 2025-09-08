# Welcome

This directory provides guidance on when and how to examine relationships between variables in research. 
Such relationships can be statistically expressed by looking at two measures: *covariance* and the *correlation coefficient*.

Topics:
* [Pearson's Correlation](https://github.com/ioakeim-h/R-for-Research/blob/main/Correlation/Pearson/Pearson.md)
* [Point-biserial Correlation](https://github.com/ioakeim-h/R-for-Research/blob/main/Correlation/Point-biserial/biserial-and-point-biserial.md)

## Covariance

The simplest way to look at whether two variables are associated is to look at whether they *covary*. A positive covariance indicates that as one variable deviates from the mean, the other variable deviates in the same direction. On the other hand, a negative covariance indicates that as one variable deviates from the mean (e.g. increases), the other deviates from the mean in the opposite direction (e.g. decreases).

The problem with covariance is that it is not a standardized measure as it is scale-dependent: its value changes if you change the units of either variable, making it hard to compare covariances in an objective way across variable pairs or datasets.

## Correlation Coefficient

The correlation coefficient is the standardized form of covariance and provides a more interpretable, unitless measure of association between variables. It is calculated by dividing the covariance by the product of the variables’ standard deviations, which scales the value to a fixed range between –1 and 1. A correlation coefficient of 1 indicates a perfect positive linear relationship, –1 a perfect negative, and 0 no linear relationship.

Generally, values around ±0.1 indicate a small effect (weak relationship), ±0.3 a medium effect (moderate relationship), and ±0.5 or greater a large effect (strong relationship). *However, these thresholds are approximate guidelines and should not replace interpretation within the specific research context and literature*.

Although we can directly interpret the effect size of a correlation coefficient, we must also test the hypothesis that the correlation is different from zero (i.e., different from “no relationship”). Significance shows whether an effect likely isn’t due to chance, while effect size shows how large or meaningful that effect is.










