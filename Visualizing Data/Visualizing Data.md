
## Visualizing Data

This is the first step of any data analysis. Begin by inspecting your data with a picture, see how it looks and only then think about interpreting the more vital statistics.

The goal of visualization is not to create delightful graphs but to present data clearly. Some general guidelines are:

-   Give axes informative labels (including units of measurement).

-   Rather than emphasizing aesthetics, induce the reader to think about the data presented.

-   Avoid excess ink: do not include features unless they are absolutely necessary to interpret the data.

-   Avoid distorting the data (e.g. by by scaling the y-axis in some weird way to give a false impression).

-   Do not use patterns, 3-D effects, shadows, etc

## Syntax for ggplots

Visual elements are known as geoms (short for geometric objects). These geoms have aesthetic properties which can be defined in general for the whole plot, or individually for a specific part of it. When aesthetics are defined for specific parts, the general settings of the plot will be overriden.

For a table outlining the parameters required for commonly used geoms, see Discovering statistics Using R textbook, page 124, table 4.1.

The attributes of the geom can be adjusted. These include: the color of the geom, the color that fills the geom, the type of line used (solid, dashed, etc), the shape of the data points (triangle, square, etc), the size of the geom, and the transparency of the geom (known as alpha).

The geoms are what make up a plot. To create a plot with ggplot, the general version of the command looks like this:

```R         
myGraph <- ggplot(myData, aes(variable_for_x_axis, variable_for_y_axis)) + geom_of_choice + labels(x = "x-axis Title", y = "y-axis Title")}
```

## Avoid Overplotting

Plots can become cluttered or unclear when there is too much data to present in a single graph, or when data on a plot overlap. To avoid this, we can make a position adjustment or use faceting.

A position adjustment can be applied by setting the position argument within a geom. Common options include:

-   "dodge" -- places elements side by side

-   "stack" and "fill" -- stack elements on top of each other (with \`"fill"\` scaling to 100%)

-   "identity" -- keeps original positions

-   "jitter" -- adds random noise to reduce overplotting

Below is an example with geom point and jitter position:

```r
ggplot(myData, aes(x, y)) + geom_point(position = "jitter")
```

Faceting splits a plot into subgroups. There are two ways to facet:

1.  **`facet_grid()`**: Creates a grid of subplots with rows and columns, specifying row and column variables.
2.  **`facet_wrap()`**: Creates a series of subplots arranged in a single row or column, wrapping into multiple rows or columns based on the number of panels.

facet_grid:

```r         
ggplot(myData, aes(x, y)) + geom_point() + facet_grid(row_var ~ col_var)
```

facet_wrap:

```r         
ggplot(myData, aes(x, y)) + geom_point() + facet_wrap(~ variable)
```

## Exploring Relationships with Scatter Plots

Scatterplots can be used to explore the relationships between variables and also help in identifying outliers. Scatterplots often have a regression line that summarizes the relationship between variables. Adding this line creates a shaded area around it which is the 95% Confidence Interval (CI).

### Simple scatter plot

The simple scatter plot looks at the relationship between two variables. The general syntax is as follows:

```r         
ggplot(myData, aes(x, y)) + 
  geom_point() + # Scattered data
  geom_smooth(method="lm") + # Adds a straight line
  labs(x="xLabel", y="yLabel") # Adds axes labels
```

-   For a line that is robust to outliers, we can use the method `"rlm"` after having loaded the "MASS" package.
-   The CI shade can be manipulated by adjusting the parameters in the `geom_smooth()` function: (1) change its colour `fill=Blue`, (2) its transparency `alpha=0.1`, (3) or switch it off `se=F`.

### Grouped scatter plot

The grouped scatterplot displays data points for multiple groups using different colours or shapes. Each group is typically defined by a categorical variable. The general syntax is as follows:

```r         
ggplot(myData, aes(x, y)) + 
  geom_point() + 
  geom_smooth(aes(fill=categorical_var), method="lm") + 
  labs(x="xLabel", y="yLabel") 
```

## Exploring Data Distributions

### Histograms

Histograms are a great way to explore the distribution of the data. In a histogram, binwidths determine how data is grouped. R gives a default value for binwidth based on calculations occurring underneath. However, we can play around with binwidth to get an idea of the data distribution. The smaller the binwidth, the more detailed the histogram is.

```r         
ggplot(myData, aes(numerical_var)) + 
  geom_histogram(binwidth = 0.4) + 
  labs(x="xLabel", y="Frequency") 
```

### Density Plots

Density plots are similar to histograms except they smooth the distribution into a line (rather than bars).

```r         
ggplot(myData, aes(x=numerical_var)) + 
  geom_density() + 
  labs(x="xLabel", y="Density Estimate") 
```

### Boxplots

Like histograms, boxplots also tell us whether the distribution is symmetrical or skewed. If the whiskers are the same length then the distribution is symmetrical; however, if the top or bottom whisker is much longer than the opposite whisker then the distribution is skewed. Data points beyond the whiskers are outliers.

The box itself represents the IQR, where the middle 50% of data lies. The lower end of the box is the Q1, where 25% of data lies. The upper end of the box is the Q3, where the remaining 25% of data lies. At the centre of the plot is the median (Q2). The whiskers extend to 1.5 times the IQR - designed to identify the range of "normal" data, thus anything beyond the whiskers is considered an outlier.

```r         
# categorical_var is optional
ggplot(myData, aes(x=categorical_var, y=numerical_var)) + 
  geom_boxplot() + 
  labs(x="xLabel", y="yLabel") 
```

## Displaying Means with Bar Charts

Bar charts are a common way for people to display means. The procedure is different depending on the number of independent variables.

### Bar charts for one independent variable

Here is a general template for creating a bar chart with error bars of 95% CI.

```r         
ggplot(myData, aes(categorical_var, numerical_var)) +
  stat_summary(fun = mean, geom = "bar", fill = "White", colour = "Black") +
  stat_summary(fun.data = mean_cl_normal, geom = "errorbar", width = 0.1) + 
  labs(x = "xLabel", y = "Mean yLabel")
```

`fun.data` takes either a normal confidence interval (mean_cl_normal) or bootstrapped (mean_cl_boot), but both require the `Hmisc` package. The argument passed to `fun.data` is what adds error bars to the chart. The geom for error bars decides the style of the error bars, and can take either `"errorbar"` or `"pointrange"`. Finally, the `width` decides the size of the error bars.

### Bar charts for multiple independent variables

The procedure is the same as before, but with the key difference that we have to specify `position = "dodge"`.

#### Plotting using aesthetics (such as colour)

```r         
ggplot(myData, aes(categorical_var, numerical_var, fill = categorical_var2)) +
  stat_summary(fun = mean, geom = "bar", position = "dodge") +
  stat_summary(fun.data = mean_cl_normal, geom = "erorbar", position=position_dodge(width=0.9), width = 0.1) +
  labs(x = "xLabel", y = "Mean yLabel", fill = "fillLabel")
```

`position = "dodge"` prevents objects from overlapping. `position_dodge(width=0.9)` makes the error bars stand side by side based on the width provided in the parenthesis. In the `labs()` function, the `fill` parameter specifies a title for the legend on the graph.

#### Plotting using facetting

This approach allows us to display the plot in different panels for each category in a variable.

```r         
ggplot(myData, aes(categorical_var, numerical_var)) +
  stat_summary(fun = mean, geom = "bar") +
  stat_summary(fun.data = mean_cl_normal, geom = "errorbar", width = 0.2) +
  facet_wrap(~ categorical_var2) + 
  labs(x="xLabel", y="yLabel") +
  theme(legend.position = "none") 
```

`theme(legend.position = "none")` removes the legend.

## Line Graphs

### Long format

Line graphs are designed to display trends or changes over time. This type of graph requires the data to be in long format, so it will not work with repeated-measures designs unless the data are restructured. To restructure the data into a long format we can use the `pivot_longer()` function:

```r         
# Line graph for single independent variable
myDataLong <- pivot_longer(myData, cols = everything(), names_to = "Variable", values_to = "Value")
```

This command gets the names of all the columns, as determined by `everything()`, and assigns them as values to the newly created column which is specified in `names_to`. The values of these columns are then assigned to the other newly created column specified in `values_to`.

When `everything()` is used, the resulting dataset has only two columns: - A categorical column (Variable) containing the original column names. - A numeric column (Value) containing the original values.

However, there are cases where there are more than one categorical variables in the resulting dataset. `everything()` works only when all columns represent measurements to be reshaped. If the dataset contains categorical or identifier variables, do **not** use `cols = everything()` because those columns should not be pivoted. Instead, exclude such variables by specifying `cols = -varToExclude`, where `varToExclude` is the name of the categorical or ID column. This tells `pivot_longer()` to reshape all columns **except** the excluded ones, keeping them as fixed identifiers.

### Factors

To plot a categorical variable in ggplot() it needs to be recognized as a factor: `myDataLong$myFactor <- as.factor(myDataLong$myVariable)`

We can rename the levels of a factor so that they appear as desired on the plot: `levels(myDataLong$myFactor) <- c("renamedLevel1","renamedLevel2")`

### Line graphs of a single independent variable

```r         
ggplot(
  myDataLong, aes(myFactor, myNumericalVar)
  
  # Create points to represent the means
) + stat_summary(fun = mean, geom = "point") +
  
  # Connect points with a line
  stat_summary(fun = mean, geom = "line", aes(group = 1), colour = "Blue", linetype = "dashed") +
  
  # Add error bars
  stat_summary(fun.data = mean_cl_boot, geom = "errorbar", width = 0.1) + 
  
  # Add labels to the axes
  labs(x = "xLabel", y= "Mean yLabel")
```

`group = 1` is used because we are joining summary points, otherwise the line will not display. `fun.data = mean_cl_boot` because the error bars on graphs of repeated-measures designs are not corrected for the fact that the data points are dependent (read section 9.2 in Discovering Statistics Using R).

### Line graphs for several independent variables

The command is the same as for a single independent variable, except that we have set the `colour` aesthetic to be the second categorical variable (e.g. Group).

```r         
ggplot(
  myDataLong, aes(myFactor, myNumericalVar, colour = Group)
) + stat_summary(fun = mean, geom = "point") +
  stat_summary(fun = mean, geom = "line", aes(group = Group)) +
  stat_summary(fun.data = mean_cl_boot, geom = "errorbar", width = 0.2) + 
  labs(x = "xLabel", y= "Mean yLabel", colour="Group") 
```
