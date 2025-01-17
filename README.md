
<!-- README.md is generated from README.Rmd. Please edit that file -->
The discreteQ repository
========================

This repository contains two different parts:

1.  In the folder repository you can find the datasets and the R codes that generated all the results in the paper "Generic Inference on Quantile and Quantile Effect Functions for Discrete Outcomes" written by Victor Chernozhukov, Ivan Fernandez-Val, Blaise Melly and Kaspar Wüthrich. This paper is available at <https://arxiv.org/abs/1608.05142>. The first empirical example uses data from the Oregon health experiment. The data are available at <http://www.nber.org/oregon/4.data.html>. The second empirical example is a decomposition of the black-white testscore gap. The data are available at <http://dx.doi.org/10.1257/aer.103.2.981>. The codes that produce the simulations results in the supplementary appendix are also provided.

2.  The R package that contain generic functions that allows researchers to easily apply the methods suggested in the paper "Generic Inference on Quantile and Quantile Effect Functions for Discrete Outcomes" written by V. Chernozhukov, I. Fernandez-Val, B. Melly, and K. Wüthrich. This paper is available at <https://arxiv.org/abs/1608.05142>. The goal of discreteQ is to perform inference on quantile functions, quantile treatment effect functions and decompositions of differences between quantile functions for possibly discrete outcomes.

Installation of the discreteQ package
=====================================

You can install the released version of discreteQ from [CRAN](https://CRAN.R-project.org) with:

``` r
# You can get the discreteQ package from GitHub:
install.packages("devtools")
devtools::install_github("bmelly/discreteQ")
```

After installing and loading the package, we recommend reading the vignette and the help file for the command discreteQ:

``` r
library(discreteQ)
vignette("discreteQ")
?discreteQ
```

Simple examples with simulated data
===================================

Example 1: confidence bands for single quantile functions
---------------------------------------------------------

`discreteQ()` can be used in three different ways. First, the command can provide uniform bands for the unconditional quantile function of an outcome variable. Let's consider an artificial example with 100 observations drawn from the Poisson distribution with mean 3:

``` r
library(discreteQ)
set.seed(1234)
outcome <- rpois(100, 3)
results1 <- discreteQ(outcome)
```

The output of the function is a list of step functions (the range of quantiles considered, the values at which the distribution function has been estimated and the type of model are also reported at then end):

``` r
results1
#> $Q
#> Step function
#> Call: stats::stepfun(F, c(ys, max(ys)), right = TRUE)
#>  x[1:8] =   0.07,   0.25,   0.51,  ...,   0.99,      1
#> 9 plateau levels =      0,      0,      1,  ...,      6,      8
#> 
#> $ub.Q
#> Step function
#> Call: stats::stepfun(lb.F.i, c(ys, max(ys)), right = FALSE)
#>  x[1:8] =      0, 0.12995, 0.36257,  ..., 0.97288,      1
#> 9 plateau levels =      0,      1,      2,  ...,      8,      8
#> 
#> $lb.Q
#> Step function
#> Call: stats::stepfun(ub.F.i, c(ys, max(ys)), right = TRUE)
#>  x[1:8] = 0.14082, 0.37005, 0.65743,  ...,      1,      1
#> 9 plateau levels =      0,      0,      1,  ...,      6,      8
#> 
#> $F
#> Step function
#> Call: stats::stepfun(ys, c(0, F))
#>  x[1:8] =      0,      1,      2,  ...,      6,      8
#> 9 plateau levels =      0,   0.07,   0.25,  ...,   0.99,      1
#> 
#> $lb.F
#> Step function
#> Call: stats::stepfun(ys, c(0, lb.F.i))
#>  x[1:8] =      0,      1,      2,  ...,      6,      8
#> 9 plateau levels =      0,      0, 0.12995,  ..., 0.97288,      1
#> 
#> $ub.F
#> Step function
#> Call: stats::stepfun(ys, c(0, ub.F.i))
#>  x[1:8] =      0,      1,      2,  ...,      6,      8
#> 9 plateau levels =      0, 0.14082, 0.37005,  ...,      1,      1
#> 
#> $q.range
#> [1] 0.05 0.95
#> 
#> $ys
#> [1] 0 1 2 3 4 5 6 8
#> 
#> $bsrep
#> [1] 200
#> 
#> $model
#> [1] "univariate"
#> 
#> $method
#> [1] "empirical"
#> 
#> attr(,"class")
#> [1] "discreteQ"
```

We recommend using the plot function to display the results:

``` r
plot(results1)
```

<img src="man/figures/README-example3-1.png" width="100%" />

The point estimates are shown in dark blue and the uniform bands in light blue. Note that, by default, `discreteQ` imposes support restrictions such that the confidence bands include only integer values in this example. See the advanced examples below for more details. It is also possible to obtain a tabular summary of the results using summary:

``` r
summary(results1)
#>       quantile QF lower bound upper bound
#>  [1,]     0.05  0           0           1
#>  [2,]     0.10  1           0           1
#>  [3,]     0.15  1           1           2
#>  [4,]     0.20  1           1           2
#>  [5,]     0.25  1           1           2
#>  [6,]     0.30  2           1           2
#>  [7,]     0.35  2           1           2
#>  [8,]     0.40  2           2           3
#>  [9,]     0.45  2           2           3
#> [10,]     0.50  2           2           3
#> [11,]     0.55  3           2           3
#> [12,]     0.60  3           2           3
#> [13,]     0.65  3           2           4
#> [14,]     0.70  3           3           4
#> [15,]     0.75  4           3           4
#> [16,]     0.80  4           3           5
#> [17,]     0.85  4           3           5
#> [18,]     0.90  5           4           5
#> [19,]     0.95  5           4           6
```

By default the increment between quantile indexes is 0.05 but this can be modified.

Example 2: confidence bands for quantile treatment effect functions
-------------------------------------------------------------------

If a treatment variable is provided in the argument `d` but the argument `decomposition` is set to `FALSE`, then `discreteQ()` provides uniform bands that cover jointly the quantile functions of the treated and control outcomes and the quantile treatment effect functions.

If regressors are provided in the argument `x`, then the conditional distribution function of the outcome given the covariates is estimated separately in the control and treated group using distribution regression. In a second step, `discreteQ()` estimates the counterfactual distributions that we would observe if all observations were not treated or treated by integrating the estimated conditional distributions over the distribution of the regressors in the whole sample. The estimated quantile treatment effect function has a causal interpretation if the treatment is randomized conditionally on the control variable (or can be treated as if it were).

Let's consider a second artificial example:

``` r
set.seed(1234)
treatment <- c(rep(0,1000), rep(1,1000))
reg <- rbinom(2000, 1, 0.4+treatment*0.2)
outcome <- rpois(2000, lambda = 2+4*reg)
```

We have generated 1000 control observations and 1000 treated observations. The binary regressor `reg` has a higher mean in treated group, which implies that the outcome has also a higher mean in the treated group. In fact, the unconditional distribution of the outcome in the treated group is stochastically dominated by the unconditional distribution of the outcome in the control group:

``` r
results2 <- discreteQ(outcome, treatment)
plot(results2, main="Difference between the unconditional quantile functions")
```

<img src="man/figures/README-example6-1.png" width="100%" /> The plotted uniform band covers the whole quantile effect function with probability 1-alpha (which is 0.95 by default); this property allows us to test several functional hypotheses. For instance, we can reject the null hypothesis that the treated and control unconditional distributions are identical because the uniform bands exclude 0 at some quantile indices. We can also reject the null hypothesis that the quantile difference is the constant because there is no value that is included in the band at all quantile indices. On the other hand, we are naturally unable to reject the null hypothesis that the quantile difference is weakly positive over the whole considered quantile range.

However, conditional on the regressors, there are no difference between the control and treated outcomes. The difference between the unconditional quantile functions should, therefore, not be interpreted as a causal effect of the treatment. This is why we now estimate the quantile treatment effect by including the regressors in the estimation:

``` r
results3 <- discreteQ(outcome, treatment, cbind(1, reg))
plot(results3)
```

<img src="man/figures/README-example7-1.png" width="100%" /> The bands now include 0 at all quantile indexes such that we can no longer reject the null hypothesis that the treatment has no effect at all.

The plot function can also be used to visualize the estimated counterfactual distributions that we would observed if nobody was treated (Q0) and that we would observe if everybody was treated (Q1). With the argument `add=TRUE` it is possible to add the results for the function Q1. With the argument `shift` it is possible to shift the bands to avoid that they overlap.

``` r
plot(results3, which="Q0")
plot(results3, which="Q1", add=TRUE, shift=0.2, col.l="dark green", col.b="light green")
```

<img src="man/figures/README-example8-1.png" width="100%" />

Example 3: confidence bands for decompositions of observed differences
----------------------------------------------------------------------

Finally, `discreteQ()` can be used to decompose the difference between the observed quantile function of the outcome for the treated group and the observed quantile function of the outcome for the control group. In order to perform this decomposition `discreteQ()` estimates the counterfactual distribution that would prevail for control observations had they had the distribution of regressors of treated observations. In other words, we integrate the conditional distribution of the control outcome given the regressors over the unconditional distribution of the regressors for the treated observations. Distribution regression is used to estimated the conditional distribution and the link function can be specified with the argument `method`.

We can illustrate this third type of applications with the same data. We simply add the argument `decomposition=TRUE` when we call `discreteQ()`:

``` r
results4 <- discreteQ(outcome, treatment, cbind(1, reg), decomposition=TRUE)
#> [1]         407  1932491451   553371328  -109713471 -1477418354   319693239
#> [7]   465455756
plot(results4)
```

<img src="man/figures/README-example9-1.png" width="100%" /> By default, `plot` shows all quantile functions in the first panel and the three elements of the decomposition in the remaining panels. Note that the uniform bands cover simultaneously all the functions that appear in the four panels.

In the top-left panel we see both observed unconditional quantile functions as well as the counterfactual quantile function. The top-right panel provides the results for the difference between the observed unconditional distribution (Q1 minus Q0). This difference is decomposed into two components that appear in the bottom panels.

The bottom-left panel provides the difference between the treated and the counterfactual quantile functions (Q1 minus Qc). These two quantile functions are obtained by integrating the same conditional distribution of the outcome over different distribution of the covariates (for Qc we use the distribution of the regressors in the control group and for Q1 we use the distribution of the covariates for the treated group). This difference is known under a variety of names in the literature: the composition effect, the explained component, the justified component, the effect of characteristics.

The bottom-right panel provides the results for the difference between the counterfactual and the control quantile functions (Qc minus Q0). These two quantile functions are obtained by integrating different conditional distribution function over the same distribution of the regressors. Thus, this part of the difference cannot be explained by the different characteristics of the groups. It is therefore called the unexplained difference. In discrimination studies, this component is often interpreted as the discrimination between the group (because it cannot be explained but this interpretation may be incorrect if important regressors are omitted or not observable).

If we can assume that the treatment is as good as conditionally randomized, then this component has a causal interpretation as the quantile treatment effect on the untreated. Section 2.3 in Chernozhukov, Fernández-Val, and Melly (2013) provides formal conditions for such an interpretation to be valid. Thus, when these assumptions are satisfied, `discreteQ()` can be used to estimate the quantile treatment effects for the whole population with the argument `decomposition = FALSE` and it can be used to estimate the quantile treatment effects on the untreated with the argument `decomposition = TRUE`. Finally, the quantile treatment effect on the treated (with a reverse sign) can be obtained with the argument `decomposition = TRUE` and `d = 1 - treatment`.

Advanced examples
=================

Continuous outcomes and support restrictions
--------------------------------------------

The methods implemented by the package `discreteQ` can be used with discrete, continuous and mixed continuous-discrete outcomes.[1] The usage of the functions is similar with different types of outcomes but the following two differences are worth noting.

First, since the estimated distribution function of the outcome jumps at all observed values of the outcome in the sample, we should in principle estimate separately the distribution function at all these thresholds. In the presence of covariates, however, estimating a large number of distribution regressions is very time consuming. For this reason, by default, `discreteQ()` will not estimate a separate distribution regression for each observed value of the outcome but will approximate the conditional distribution using a grid of 100 values. This default behavior can be modified with the argument `ys`. For instance, setting `ys = Inf` ensures that the distribution function will be estimated at all observed values.

Second, as shown in Chernozhukov et al. (2019), when the outcome is discrete we can make the QF-bands and QE-bands more informative by imposing support restrictions. By default, the function `plot.discreteQ()` assumes that the support of the outcome is discrete when the empirical support contains less than 200 different values. In this case, the true support is estimated by the empirical support of the outcome. For continuous outcomes, the user should override this default behavior by setting `support = "continuous"` when calling the `plot` function for `discreteQ` objects.

We know illustrate the use of the `discreteQ` functions with a continuous outcome:

``` r
set.seed(1234)
outcome <- rnorm(500, 3)
results5 <- discreteQ(outcome, ys = Inf)
plot(results5, support = "continuous")
```

<img src="man/figures/README-example10-1.png" width="100%" />

Estimation of conditional distribution functions in the presence of covariates
------------------------------------------------------------------------------

In the presence of covariates, the conditional distribution function of the outcome given the covariates must be estimated. The main estimation method implemented by `discreteQ` is distribution regression. This method consists in estimating a different binary regression for each threshold *y*. The binary variable 1(*y*<sub>*i*</sub> ≤ *y*) is regressed on the regressors `x`. This yields the distribution regression model with
*F*<sub>*Y* ∣ *X*</sub>(*y*∣*x*) = *P*(*Y* ≤ *y* ∣ *X* = *x*)=*Λ*<sub>*y*</sub>(*x*<sup>′</sup>*β*(*y*))
 where *Λ*<sub>*y*</sub>(⋅) is a known link function which is allowed to change with the threshold level *y* and *β*(*y*) is an unknown vector of parameters.

By default, `discreteQ()` uses the logistic link function *Λ*(*x*′*β*(*y*)) = 1/(1 + *e*<sup>−*x*′*β*(*y*)</sup>). Alternatively, the Gaussian link function is used if `method = "probit"`, the complementary log-log link function (which nests as a special case the Cox proportional hazard model) if `method = "cloglog"`, the Cauchy link function if `method = "cauchit"`, the Poisson link function suggested by Chernozhukov et al. (2019) (which nests as a special case the Poisson regression model) if `method = "drp"`, the identity link function (which consists in estimating an OLS regression for each threshold, also called the linear probability model) if `method = "lpm"`.

The fully parametric Poisson regression model is also implemented by `discreteQ()` when `method = "poisson"`. The difference between the Poisson regression model and the distribution regression models discussed above is that the Poisson regression model imposes the same index *x*′*β* at all thresholds. This is a strong homogeneity assumption which may be rejected by the data even when there are no covariates or a fully saturated set of covariates. On the other hand, the distribution regression model allows for a different vector of coefficients *β*(*y*) at each threshold. This model does not impose any parametric assumption on the data when we have a fully saturated set of covariates. The distribution regression with the Poisson link function (implemented when `method = "drp"`) strictly nests the Poisson regression model (implemented when `method = "poisson"`). If these two methods give statistically different results then we must conclude that the Poisson regression model is rejected by the data.

Weighted cluster bootstrap
--------------------------

By default, the function `discreteQ()` assumes that the sample has been randomly drawn from the population of interest. In this case, the (Bayesian) weighted bootstrap is used to estimate the uniform bands for the distribution functions: For each observation and each replication the function randomly draw a weight from the standard exponential distribution. For each replication, the distribution function is estimated using the original but weighted sample. An advantage of this method (compared with the nonparametric bootstrap) is that no bootstrap sample will suffer from multicollinearity if the original sample does not suffer from multicollinearity.

Cluster sampling gives rise to correlation between units within the same cluster. For example, the number of doctor visits of individuals within a household may be correlated because of common (often unobserved) characteristics of individuals within a household or because of features of the household itself (such as type of retirement plan). When this is the case, the user must provide a vector that identifies the clusters to the argument `cluster`. When this is the case, the weighted cluster bootstrap is used. The weights are still draw from the standard exponential distribution but all the units in the same cluster receive the same weight. This procedure allows for abitrary correlation within the clusters but assume random sampling of the clusters. The number of clusters must be large.

Parallel computing
------------------

The computing time may be long if the numbers of observations, regressors, points in the support of the dependent variable, and bootstrap replications are large. For this reason, parallel processing has been implemented by `discreteQ()`. The user must first create a cluster object with the desired number of parallel processes. This object is then pass to `discreteQ()`via the argument `cl`. Here is a comparison of the computing time for a simple example:

``` r
set.seed(1234)
treatment <- c(rep(0,1000), rep(1,1000))
reg <- rbinom(2000, 1, 0.4+treatment*0.2)
outcome <- rpois(2000, lambda = 2+4*reg)
#Without parallel computing
set.seed(42)
system.time(results6 <- discreteQ(outcome, treatment, reg))
#>    user  system elapsed 
#>   16.82    0.03   16.86
my_cl <- parallel::makePSOCKcluster(2)
#With parallel computing
set.seed(42)
system.time(results7 <- discreteQ(outcome, treatment, reg, cl = my_cl ))
#>    user  system elapsed 
#>    0.13    0.04    9.03
```

The results are replicable simply by setting the seed with `set.seed()`. If this is done, exactly the same results will be obtained with or without parallel computing. In both cases, a sequence of random streams (a seed for each bootstrap replication) is generated with `rngtools:RNGseq`. This sequence is returne by `discreteQ()` if `return.seeds = TRUE`. Such a sequence can be passed to the argument `list_of_seeds` of `discreteQ()`, which permits replicating particular bootstrap draws.

``` r
#Results with and without parallel computing are equal
all.equal(results6, results7)
#> [1] TRUE
```

Changing the confidence level or the quantile range
---------------------------------------------------

The confidence bands estimated by `discreteQ()` cover the true functions in the range of quantiles determined by the argument `q.range` with the coverage rate equal to 1 minus the confidence level `alpha`. If the user wants to obtain the bands for different quantile ranges or different confidence levels, she can save the bootstrap results by setting the argument `return.boot = TRUE` and then provide these bootstrap results to the argument `old.res`. This avoids computing the results each time when the only difference is the level or the quantile range. Here is an example:

``` r
#95% confidence bands (default value)
results8 <- discreteQ(outcome, treatment, reg, return.boot = TRUE)
#90% confidence bands
results9 <- discreteQ(outcome, treatment, reg, old.res = results8, alpha = 0.1)
```

Concluding remarks
==================

For the precise definition of the estimators and their statistical properties, you are invited to read the paper "Generic Inference on Quantile and Quantile Effect Functions for Discrete Outcomes" written by Victor Chernozhukov, Ivan Fernandez-Val, Blaise Melly, and Kaspar Wüthrich. This paper is available at <https://arxiv.org/abs/1608.05142>.

References
==========

Chen, Mingli, Victor Chernozhukov, Iván Fernández-Val, and Blaise Melly. 2017. “Counterfactual: An R Package for Counterfactual Analysis.” *The R Journal* 9 (1): 370–84. doi:[10.32614/RJ-2017-033](https://doi.org/10.32614/RJ-2017-033).

Chernozhukov, Victor, Ivan Fernandez-Val, Blaise Melly, and Kaspar Wüthrich. 2019. “Generic Inference on Quantile and Quantile Effect Functions for Discrete Outcomes.” *arXiv Preprint*. <https://arxiv.org/abs/1608.05142>.

Chernozhukov, Victor, Iván Fernández-Val, and Blaise Melly. 2013. “Inference on Counterfactual Distributions.” *Econometrica* 81 (6). Wiley Online Library: 2205–68.

[1] For continuous outcomes we nevertheless recommend using the package `Counterfactual`, which is available from CRAN (Chen et al. 2017). Its functions offers additional options and sharper inference for quantile effects when the outcome is continuous.
