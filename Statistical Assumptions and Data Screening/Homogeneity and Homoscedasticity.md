## Homogeneity and Homoscedasticity

Homogeneity of variance (or homoscedasticity) means that the variances (squared standard deviations) of the scores in different groups or across different conditions are approximately equal. Put in another way, the assumption asserts that as we go through levels of one variable, the variance of the other should not change. The two terms mean essentially the same thing — equal variances — but are used in different contexts:

-   **Homogeneity of variance**: used when comparing groups (e.g. ANOVA). If we have collected groups of data (i.e. categorical data) then this means that the variance of the outcome variable or variables should be the same in each of these groups.

-   **Homoscedasticity**: used in regression or continuous predictors describing equal variance of residuals. If we have collected continuous data (such as in correlational designs), this assumption means that the variance of one variable should be stable at all levels of the other variable.

### Testing homogeneity of variance

While visual methods such as boxplots are commonly used to assess homogeneity of variance, they have limitations in formal research, which is why statistical tests are often prioritized in publications. The key difference between visual checks for homogeneity of variance and for homoscedasticity is that the former examines variance across categorical groups, while the latter examines variance across continuous values. Because there is no objective threshold for what constitutes “equal enough” variance, visual assessments of homogeneity of variance are inherently subjective. Nevertheless, they remain useful for identifying groups that may warrant closer examination.

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
