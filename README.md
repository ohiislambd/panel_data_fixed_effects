Panel Data and Fixed Effects: An Intuition and Simulation Tutorial
================
Ohi Islam
Spring 2026

- [Introduction](#introduction)
- [What Is Panel Data?](#what-is-panel-data)
  - [Example Structure](#example-structure)
- [Panel Data Model](#panel-data-model)
- [OLS Assumptions](#ols-assumptions)
  - [What Happens if This Fails?](#what-happens-if-this-fails)
- [Fixed Effects Model](#fixed-effects-model)
- [The Demeaning Transformation](#the-demeaning-transformation)
- [Simulation Exercise](#simulation-exercise)
- [Step 1: Simulate Panel Data](#step-1-simulate-panel-data)
- [Step 2: Pooled OLS](#step-2-pooled-ols)
- [Step 3: Demeaning](#step-3-demeaning)
- [Step 4: Fixed Effects Regression](#step-4-fixed-effects-regression)
- [Step 5: Compare Estimates](#step-5-compare-estimates)
- [Visualization](#visualization)
- [Key Takeaways](#key-takeaways)
- [Practice Questions](#practice-questions)
- [Conclusion](#conclusion)

# Introduction

This tutorial introduces the **panel data fixed effects model** using
intuition and simulation.  
The goal is to help students understand:

- what panel data are  
- how fixed effects models work  
- why pooled OLS can be biased  
- how the **demeaning (within) transformation** solves the problem

We will simulate data where **parental education influences GPA** and is
correlated with a **merit-based scholarship**. We will show that:

1.  pooled OLS produces a biased estimate
2.  the fixed effects estimator recovers the true treatment effect

------------------------------------------------------------------------

# What Is Panel Data?

Panel data follow the **same units over multiple time periods**.

Examples of units include:

- students  
- households  
- firms  
- schools

Because we observe the same individuals repeatedly, panel data allow us
to study **changes within individuals over time**.

------------------------------------------------------------------------

## Example Structure

``` r
example_panel <- data.frame(
  student_id = c(1,1,1,2,2,2),
  semester = c(1,2,3,1,2,3),
  GPA = c(3.1,3.3,3.4,2.8,2.9,3.0),
  scholarship = c(0,1,1,0,0,1),
  parental_education = c(16,16,16,12,12,12)
)

knitr::kable(
  example_panel,
  digits = 2,
  align = "c",
  caption = "Example Structure of Panel Data: Each Student Appears Multiple Times"
)
```

| student_id | semester | GPA | scholarship | parental_education |
|:----------:|:--------:|:---:|:-----------:|:------------------:|
|     1      |    1     | 3.1 |      0      |         16         |
|     1      |    2     | 3.3 |      1      |         16         |
|     1      |    3     | 3.4 |      1      |         16         |
|     2      |    1     | 2.8 |      0      |         12         |
|     2      |    2     | 2.9 |      0      |         12         |
|     2      |    3     | 3.0 |      1      |         12         |

Example Structure of Panel Data: Each Student Appears Multiple Times

Here:

- `student_id` identifies individuals  
- `semester` identifies time  
- `GPA` is the outcome  
- `scholarship` varies over time  
- `parental_education` is time-invariant

------------------------------------------------------------------------

# Panel Data Model

Let:

- $i$ index individuals  
- $t$ index time

A basic regression is

$$
Y_{it} = \beta_0 + \beta_1 X_{it} + u_{it}
$$

Panel models often include **unobserved individual effects**:

$$
Y_{it} = \beta_0 + \beta_1 X_{it} + \alpha_i + \varepsilon_{it}
$$

where

- $Y_{it}$ = GPA  
- $X_{it}$ = scholarship indicator  
- $\alpha_i$ = individual-specific constant (family background,
  ability)  
- $\varepsilon_{it}$ = idiosyncratic error

------------------------------------------------------------------------

# OLS Assumptions

OLS requires

$$
E[u_{it} | X_{it}] = 0
$$

This means regressors must be **uncorrelated with the error term**.

------------------------------------------------------------------------

## What Happens if This Fails?

If an omitted variable affects the outcome **and** is correlated with
the regressor, OLS suffers from **omitted variable bias**.

Example:

- parental education affects GPA
- parental education also affects scholarship probability

A regression of GPA on scholarship alone may incorrectly attribute
parental education effects to scholarship.

------------------------------------------------------------------------

# Fixed Effects Model

The fixed effects model is

$$
GPA_{it} = \beta_0 + \beta_1 Scholarship_{it} + \alpha_i + \varepsilon_{it}
$$

If $\alpha_i$ is correlated with scholarship, pooled OLS is biased.

The fixed effects estimator removes $\alpha_i$.

------------------------------------------------------------------------

# The Demeaning Transformation

Take the time average for individual $i$:

$$
\bar{Y}_i = \beta_0 + \beta_1 \bar{X}_i + \alpha_i + \bar{\varepsilon}_i
$$

Subtract from the original equation:

$$
Y_{it}-\bar{Y}_i = \beta_1(X_{it}-\bar{X}_i) + (\varepsilon_{it}-\bar{\varepsilon}_i)
$$

The individual effect $\alpha_i$ disappears.

This is called the **within transformation** or **demeaning**.

------------------------------------------------------------------------

# Simulation Exercise

We now simulate a dataset where:

- parental education affects GPA
- parental education is correlated with scholarship
- scholarship has a true causal effect

------------------------------------------------------------------------

# Step 1: Simulate Panel Data

``` r
set.seed(123)

N <- 500
T <- 4

beta0 <- 2
beta1 <- 0.2
beta2 <- 0.05

student_id <- rep(1:N, each=T)
semester <- rep(1:T, times=N)

parental_education <- rep(rnorm(N,14,2), each=T)

alpha_i <- rep(rnorm(N,0,0.3), each=T)

latent <- -1 + 0.4*parental_education + 0.7*alpha_i + rnorm(N*T)

prob <- plogis(latent)

scholarship <- rbinom(N*T,1,prob)

epsilon <- rnorm(N*T,0,0.2)

gpa <- beta0 + beta1*scholarship + beta2*parental_education + alpha_i + epsilon

panel_df <- data.frame(
  student_id,
  semester,
  gpa,
  scholarship,
  parental_education
)

head(panel_df)
```

    ##   student_id semester      gpa scholarship parental_education
    ## 1          1        1 2.702695           1           12.87905
    ## 2          1        2 2.793407           1           12.87905
    ## 3          1        3 2.797585           1           12.87905
    ## 4          1        4 2.406553           1           12.87905
    ## 5          2        1 2.173651           1           13.53965
    ## 6          2        2 3.019938           1           13.53965

The $true$ regression model should include $parental\ education$ as a
regressor in the model. Let us run the true model

``` r
head(panel_df)
```

    ##   student_id semester      gpa scholarship parental_education
    ## 1          1        1 2.702695           1           12.87905
    ## 2          1        2 2.793407           1           12.87905
    ## 3          1        3 2.797585           1           12.87905
    ## 4          1        4 2.406553           1           12.87905
    ## 5          2        1 2.173651           1           13.53965
    ## 6          2        2 3.019938           1           13.53965

``` r
model1<-lm(gpa ~ scholarship+parental_education+factor(student_id), data=panel_df)
coef(model1)[c("scholarship","parental_education")]
```

    ##        scholarship parental_education 
    ##         0.23174935         0.06210167

------------------------------------------------------------------------

# Step 2: Pooled OLS

``` r
pooled_model <- lm(gpa ~ scholarship, data=panel_df)

summary(pooled_model)
```

    ## 
    ## Call:
    ## lm(formula = gpa ~ scholarship, data = panel_df)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -1.1737 -0.2537  0.0022  0.2406  1.2281 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value             Pr(>|t|)    
    ## (Intercept)  2.51363    0.06607  38.047 < 0.0000000000000002 ***
    ## scholarship  0.39235    0.06658   5.892        0.00000000445 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.3678 on 1998 degrees of freedom
    ## Multiple R-squared:  0.01708,    Adjusted R-squared:  0.01659 
    ## F-statistic: 34.72 on 1 and 1998 DF,  p-value: 0.000000004454

This regression omits parental education, which is a determinant of GPA
and is also correlated with scholarship receipt. Because this omitted
variable is positively correlated with both the outcome and the
treatment variable, the estimated scholarship coefficient suffers from
upward omitted variable bias. The regression therefore attributes part
of the effect of parental education to the scholarship variable, leading
to an overestimate of the true treatment effect.

------------------------------------------------------------------------

# Step 3: Demeaning

``` r
demeaned_df <- panel_df %>%
  group_by(student_id) %>%
  mutate(
    gpa_dm = gpa - mean(gpa),
    scholarship_dm = scholarship - mean(scholarship),
    parental_dm = parental_education - mean(parental_education)
  ) %>%
  ungroup()
```

Check parental education after demeaning.

``` r
summary(demeaned_df$parental_dm)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##       0       0       0       0       0       0

Because parental education is constant within students, its demeaned
value is zero.

------------------------------------------------------------------------

# Step 4: Fixed Effects Regression

``` r
fe_model <- lm(gpa_dm ~ scholarship_dm - 1, data=demeaned_df)

summary(fe_model)
```

    ## 
    ## Call:
    ## lm(formula = gpa_dm ~ scholarship_dm - 1, data = demeaned_df)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -0.51757 -0.11879 -0.00054  0.11581  0.63155 
    ## 
    ## Coefficients:
    ##                Estimate Std. Error t value       Pr(>|t|)    
    ## scholarship_dm  0.23175    0.03636   6.375 0.000000000227 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.1715 on 1999 degrees of freedom
    ## Multiple R-squared:  0.01992,    Adjusted R-squared:  0.01943 
    ## F-statistic: 40.64 on 1 and 1999 DF,  p-value: 0.0000000002272

------------------------------------------------------------------------

# Step 5: Compare Estimates

``` r
pooled_est <- coef(pooled_model)["scholarship"]

fe_est <- coef(fe_model)["scholarship_dm"]

comparison <- data.frame(
  Model=c("True Effect","Pooled OLS","Fixed Effects"),
  Estimate=c(beta1, pooled_est, fe_est)
)

comparison
```

    ##                        Model  Estimate
    ##                  True Effect 0.2000000
    ## scholarship       Pooled OLS 0.3923452
    ## scholarship_dm Fixed Effects 0.2317493

------------------------------------------------------------------------

# Visualization

``` r
ggplot(comparison, aes(x=Model,y=Estimate)) +
  geom_col() +
  theme_minimal() +
  labs(
    title="Comparing Scholarship Effect Estimates",
    y="Estimated Effect"
  )
```

![](panel-data-FE-model_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->
The simulation illustrates that the coefficient on scholarship obtained
from the demeaned fixed effects model is close to the true treatment
effect, while the pooled OLS estimate is larger. The upward bias in the
pooled regression arises because parental education affects GPA and is
positively correlated with scholarship assignment. Since parental
education is omitted from the pooled model, its effect is partially
captured by the scholarship coefficient. The fixed effects estimator
removes this bias by eliminating all time-invariant individual factors
through the within (demeaning) transformation. Consequently, even when
parental education is unobserved or difficult to measure in survey data,
the fixed effects approach can still control for its influence and
produce a more reliable estimate of the treatment effect.

------------------------------------------------------------------------

# Key Takeaways

1.  Panel data track the same individuals over time.
2.  Omitted time-invariant variables can bias OLS estimates.
3.  Fixed effects remove time-invariant heterogeneity.
4.  The demeaning transformation eliminates individual-specific
    constants.
5.  Fixed effects recover the true treatment effect when omitted
    variables are constant over time.

------------------------------------------------------------------------

# Practice Questions

1.  Why does pooled OLS produce bias in this simulation?
2.  What assumption of OLS is violated?
3.  Why does demeaning remove parental education?
4.  What types of omitted variables can fixed effects remove?
5.  What types of omitted variables remain a problem?

------------------------------------------------------------------------

# Conclusion

Panel data allow researchers to control for unobserved individual
characteristics that remain constant over time. The fixed effects
estimator achieves this by removing those time-invariant components
through the within transformation. In this simulation, pooled OLS
produces a biased estimate of the scholarship effect, while the fixed
effects estimator recovers the true causal effect more accurately.
