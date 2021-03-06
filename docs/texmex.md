# Marginal distribution and generalized Pareto tails



:::keyidea

* Fit marginal models with generalized Pareto tails and empirical distribution for the body of the data.
* Pick suitable thresholds for generalized Pareto distribution using diagnostic plots

:::

The conditional extremes model attempts to describe the general behaviour of a $D$ random vector $\boldsymbol{Y}$ when component $Y_j$ exceeds a high threshold $u_j$. This parallels other multivariate peaks-over-threshold models, including the multivariate generalized Pareto distributions arise as the limiting distribution given that $\max_{j=1}^D Y_j > u_j$ under an hypothesis of regular variation and the $R$-Pareto processes when $R(\boldsymbol{Y})>u$ for a one-homogeneous functional $R$. 

Univariate extreme value theory suggests that $Y_j \mid Y_j > u_j$ can be approximated by a generalized Pareto distribution with scale $\tau$ and shape $\xi$ above $u_j$ and we can proceed likewise for other components of $\boldsymbol{Y}$. However, it is worth keeping in mind that the other components of $\boldsymbol{Y}$ need not be simultaneously large when $Y_j>u_j$. Rather than specify a parametric model for $Y_k$ $(k \neq j)$, we can use the empirical distribution,
\begin{align*}
F^{(k)}_n(y) = n^{-1}\sum_{i=1}^n \mathrm{I}(y \leq y_{ik})
\end{align*}
or a semiparametric alternative
\begin{align*}
\widehat{F}_k(y) = 
\begin{cases}
F{(k)}_n(y) & y \leq u_k; \\
1-\{1-F^{(k)}_n(u_k)\}\left(1+\xi_k\frac{y-u_k}{\tau_k}\right)^{-1/\xi_k}_{+} & y > u_k.
\end{cases}
\end{align*}
to model the marginal distribution. This leaves the matter of the dependence structure.

## Threshold selection 

To determine which threshold to select for marginal exceedances, we can produce threshold stability plots: these consist of shape estimates (with pointwise 95% confidence intervals, even if these are presented as bands) over a range of thresholds $u_1 < \cdots < u_m$. Assuming the generalized Pareto approximation to be adequate at $u_l$, the estimates should stabilize for thresholds $u_{l+1}, \ldots, u_m$ [@Davison:1990].


```r
thstab <- 
  with(FortCollins,
     texmex::gpdRangeFit(
       data = precip,
       umin = 20,
       umax = 50,
       nint = 10L,
       penalty = "none") 
     # alternative is 'gaussian', yielding MAP
)
# there are two methods for graphics: 
# plots (base R) or ggplot (ggplot2)
patchwork::wrap_plots(ggplot(thstab))
```

<div class="figure" style="text-align: center">
<img src="texmex_files/figure-html/threshStabPlots-1.png" alt="Threshold stability plots for adjusted scale (left) and shape parameter (right) for the Fort Collins daily cumulated precipitation." width="85%" />
<p class="caption">(\#fig:threshStabPlots)Threshold stability plots for adjusted scale (left) and shape parameter (right) for the Fort Collins daily cumulated precipitation.</p>
</div>


While the shape estimates decrease steadily as we increase the threshold, it seems here that 25mm is a reasonable marginal threshold for rainfall.

The function `mrl` produces mean residual life plots `ggplot(texmex::mrl(data = precip))`, but the literature on graphics suggests the human eye has a much easier time detecting lack of trend than linearity.

::: yourturn

- Repeat the exercise, this time with daily minimum and maximum temperature. 
- For the chosen threshold, fit a generalized Pareto using the function `evm` with threshold `th`. You can explore the different methods (maximum likelihood, penalized maximum likelihood, Bayesian estimation with Markov chain Monte Carlo using independent elliptical proposals)
- Produce a quantile-quantile plot to check the goodness of fit (via `plot` or `ggplot` for the object returned by `evm`).

:::

## Standardization of the margins

The Heffernan-Tawn model describes extremes for standardized data, but there is a direct correspondence between extremes $\{\boldsymbol{Y}: Y_j > u_j\}$ and $\{\boldsymbol{Y}: t_j(Y_j) > t_j(u_j)\}$ for a monotone transformation $t_j$. The standardization in @Heffernan:2004 is to the Gumbel scale, but subsequent work [@Keef:2013b] suggests a better choice is standard Laplace, i.e., a distribution with double exponential tails that can capture negative and positive dependence.

The standard Laplace distribution has density $f(x) = \exp(-|x|)/2$ on $\mathbb{R}$ and distribution function
\begin{align*}
F(x) = \begin{cases}
\frac{1}{2}\exp(x) & x < 0, \\
1-\frac{1}{2}\exp(-x) & x \geq 0.
\end{cases}
\end{align*}
Thus, if $Y_1, \ldots, Y_D$ is a random vector with marginal distribution functions $F_1, \ldots, F_D$, we can make the margins standard Laplace by applying the quantile transformation
\begin{align*}
t_k(Y_k) = 
\begin{cases}
\log\{2{F}_k(Y_k)\} & {F}_k(Y_k) < 1/2, \\
-\log[2\{1-{F}_k(Y_k)\}] & {F}_k(Y_k) \geq 1/2,
\end{cases}
\qquad k=1, \ldots, D.
\end{align*}
In practice, we replace the unknown $k$th marginal distribution function $F_k$ by the semiparametric estimator $\widehat{F}_k$.

# Multivariate model

:::keyidea

* The multivariate conditional extremes model is a semiparametric regression model given one component is large.
* The model description assumes data are standardised to unit Laplace scale.
* Parameter estimates are obtained under a working assumption of normality and independence between components.
* Diagnostic plots for the models: 
	- threshold stability for dependence parameters
	- tail correlation $\chi(u)$: agreement between empirical estimates and fitted $\chi(u)$ curve
	-  regression plots for conditional independence of residuals. 

:::


The conditional multivariate extreme model of @Heffernan:2004 is of the form 
\begin{align*}
t(\boldsymbol{Y}_{-j})\mid t_j(Y_j)=t_j(y_j)  \approx \boldsymbol{\alpha}_{|j}t_j(y_j)+t_j(y_j)^{\boldsymbol{\beta}_{|j}}\boldsymbol{Z}
\end{align*}
with $\boldsymbol{\alpha}_{|j} \in [-1,1]^{D-1}$ and $\boldsymbol{\beta}_{|j} \in [-\infty, 1]^{D-1}$ and $\boldsymbol{Z}$ are unspecified residuals.
 
We fit the model under the working assumption that $\boldsymbol{Z} \sim \mathsf{No}_{D-1}(\boldsymbol{\mu}_{|j}, \mathrm{diag}\{\boldsymbol{\sigma}_{|j}^2\})$ with **nuisance parameters** $\boldsymbol{\mu}_{|j}$ and $\boldsymbol{\sigma}^2_{|j}$. Since each margin is conditionally independent of the others, we can break the optimization in smaller chunks, by estimating each pair $a_{k|j}, b_{k|j}$, etc. Thus, we fit each of the $D-1$ margins separately with the likelihood derived from 
\begin{align*}
t_k(Y_k) \mid t(Y_j)=t_j(y_j) \sim \mathsf{No}\left(\alpha_{k|j}t_j(y_j) + t_j(y_j)^{\beta_{k|j}}\mu_{i|j}, t_j(y_j)^{2\beta_{k|j}}\sigma^2_{k|j}\right), \qquad k \neq j,
\end{align*}
The profile likelihood for the pair ($a_{k|j}$, $b_{k|j}$) is easily obtained upon noting that, for the data \[
z_{ik} =\frac{t_k(y_{ik}) - \alpha_{k|j}t_j(y_{ij})}{t_j(y_{ij})^{\beta_{k|j}}},
\]
the conditional maximum likelihood estimators of $\mu_{k|j}$ and $\sigma^2_{k|j}$ are $\widehat{\mu}_{k|j} \mid (\alpha_{i|j}, \beta_{i|j}) = \overline{z}_{k}$ and $\widehat{\sigma}^2_{k|j}\mid (\alpha_{i|j}, \beta_{i|j}) = n^{-1}\sum_{i=1}^n (z_{ik} - \overline{z}_{k})^2$.


## Strength of extremal dependence

Before fitting the Heffernan-Tawn model, we can look at the strength of the extremal dependence between pairs. 

We can first calculate the coefficient of tail dependence $\chi$ and $\overline{\chi}$ to investigate the strength of the extremal dependence.

The tail correlation coefficient $\chi(u)$ is
\[\chi(v)= \frac{\Pr\{F_1(Y_1) > v, F_2(Y_2)>v\}}{1-v}.
\]
and if $\chi = \lim_{v \to 1} \chi(v) >0$, we say that $(Y_1, Y_2)$ are asymptotically dependent. Since $\chi=0$ for all asymptotically independent process, this is not useful measure. 

The tail dependence coefficient [@Coles:1999] $\bar{\chi}$ is estimated using
\begin{align*}
\bar{\chi}(v) = \frac{2\log(1-v)}{\log[\Pr\{F_1(Y_1) > v, F_2(Y_2) > v\}}-1
\end{align*}
for $v \in (0,1)$, so $\overline{\chi} \in [-1,1]$ and asymptotically dependent processes have $\lim_{v \to 1} \overline{\chi}(v)=1$.  The statistic $\chi$ is only useful when the data are asymptotically dependent, so we normally look first at $\overline{\chi}$ and if the Wald confidence intervals include 1, look at the strength of the dependence using the plot of $\chi$. The `texmex` package will gray out the latter if asymptotic dependence is ruled out. 


```r
# Create plot with matrix or data frame
#  with 2 columns only
chiplot_13 <- 
  chi(FortCollins[,c("maxTemp","precip")])
chiplot_23 <- 
  chi(FortCollins[,c("minTemp","precip")])

# Truncated the plots to focus on the upper right tail
# because most of the rainfall records are zero
ggplot(chiplot_13, xlim = c(0.9,1))
ggplot(chiplot_23, xlim = c(0.9,1))
```

<div class="figure" style="text-align: center">
<img src="texmex_files/figure-html/chiPlots-1.png" alt="Tail dependence plots for maximum daily temperature (top panel) and minimum daily temperature (bottom panel), conditional on extreme rainfall. The right-hand panel is greyed out if asymptotic dependence has been ruled out." width="85%" /><img src="texmex_files/figure-html/chiPlots-2.png" alt="Tail dependence plots for maximum daily temperature (top panel) and minimum daily temperature (bottom panel), conditional on extreme rainfall. The right-hand panel is greyed out if asymptotic dependence has been ruled out." width="85%" />
<p class="caption">(\#fig:chiPlots)Tail dependence plots for maximum daily temperature (top panel) and minimum daily temperature (bottom panel), conditional on extreme rainfall. The right-hand panel is greyed out if asymptotic dependence has been ruled out.</p>
</div>

If the conditional extremes model holds, then $t_k(Y_{k}) \mid t_j(Y_{j}) > t_j(u_j)$ can be approximated by $\widetilde{Y}^*_{k|j} \approx \alpha_{k|j}t_j(Y_j)+t_j(Y_j)^{\beta_{k|j}}Z_{k|j}$. We thus obtain a formula for $\chi(v)$ above the dependence threshold $u^{\star}=t_j(u)$, where for any $v^{\star}=-\log\{2\cdot(1-v)\}$ $(v > 0.5)$ such that $v^{\star} > u^{\star}$, we have
\begin{align}
\chi(v) = \tfrac{\Pr\{t_j(Y_j) > u^{\star}\}\Pr\{\widetilde{Y}^{\star}_{k|j} > v^{\star}, t_j(Y_j) > v^{\star}, t_j(Y_j) > v^{\star} \mid t_j(Y_j) > u^{\star}\}}{\Pr\{t_j(Y_j) > v^{\star}\}} \label{HTchi}
\end{align}
We can estimate empirically the coefficient of tail dependence by resampling values of $Z_{k|j}$. The values are simulation-based, so we can use the average over many replications. This does not factor into account the parameter uncertainty, which would require using the bootstrap.

The `texmex` package also includes a multivariate conditional estimator of Spearman's $\rho$ (linear correlation on the scale of $F_1(Y_1), \ldots, F_D(Y_D)$).
The function `MCS` computes the estimator, whereas `bootMCS` can be used to get a measure of uncertainty by running a nonparametric bootstrap and computing 95% pointwise confidence intervals.

::: yourturn

- Repeat the exercise with the `minTemp`/`maxTemp` pair.
- What do these sets of plots suggest about the asymptotic regime (asymptotic independence or dependence) and the strength of the dependence?

:::


## Fitting the multivariate model

The main function for fitting the conditional extreme value model in `texmex` is `mex` which gives the two stage procedure
and calls internally.

- First, `migpd` fits a generalized Pareto distribution to each margin. Users must provide 
     - `mth` or `mqu`: either the  thresholds $u_1, \ldots, u_D$ (argument `mth`) or else a vector of probability levels $F_1(u_1), \ldots, F_D(u_D)$
 (argument `mqu`)
     - User can compute the maximum a posteriori (instead of the maximum likelihood estimates) for parameters, with $\log(\tau) \sim \mathsf{No}(0,100)$ independent of $\xi \sim \mathsf{No}(0,0.25)$ as default prior specifications if `penality='gaussian'` (the default).
- These standardized data are fed to `mexDependence`. The latter has multiple arguments: 
    - `which`: the column index for the conditioning variable 
    - `dqu`: the probability level of the marginal quantile at which to specify $t(\boldsymbol{Y}) \mid t_j(Y_j) > t_j(u_j)$; note that this threshold need not be the same as the marginal threshold for the generalized Pareto. 
    - `margins` the choice of distribution for the standardisation (default is 'laplace')
    - `marTransform`: how to model $F_k$ to transforms every component of $\boldsymbol{Y}$ to standard Laplace margins. The default, `marTransform="mixture"` use the semiparametric transformation (the data are modelled with the empirical distribution below the threshold and with a generalized Pareto above $u$). The alternative is to use the empirical distribution (`marTransform = "empirical"`).
    - `constrain` whether to impose constraints to improve self-consistency for estimates of $\boldsymbol{a}_{|j}$ and $\boldsymbol{b}_{|j}$; defaut is `TRUE`;
    - `PlotLikDo` logical; whether to produce a bivariate profile likelihood defined on a curved subset of $[-1,1] \times[-\infty, 1]$ if `constrain=TRUE`.
  
In general, unless you want to obtain the profile likelihood plot, you could directly use `mex` which combines these two steps.


Once we have determined marginal and functional thresholds and the choice of conditioning variable, we can perform all steps at once using `mex`.


```r
marg <- 
  FortCollins %>% 
  select(!date) %>% 
  migpd(
    mqu = rep(0.9, 3), 
    penalty = 'none'
    )
condModFit <- 
  mexDependence(
    x = marg,
    which = "precip", 
    dqu = 0.9, 
    PlotLikDo = FALSE,
    PlotLikRange = list(a = c(-1, 1),
                        b = c(-3, 1))
    )
print(condModFit)
#> mexDependence(x = marg, which = "precip", dqu = 0.9, PlotLikDo = FALSE, 
#>     PlotLikRange = list(a = c(-1, 1), b = c(-3, 1)))
#> 
#> 
#> Marginal models:
#> 
#> Dependence model:
#> 
#> Conditioning on precip variable.
#> Thresholding quantiles for transformed data: dqu = 0.9
#> Using laplace margins for dependence estimation.
#> Constrained estimation of dependence parameters using v = 10 .
#> Log-likelihood = -5996 -6160 
#> 
#> Dependence structure parameter estimates:
#>   maxTemp minTemp
#> a  0.0201   0.160
#> b -0.6488  -0.401
```

<img src="texmex_files/figure-html/fit_mex-1.png" width="85%" style="display: block; margin: auto;" /><img src="texmex_files/figure-html/fit_mex-2.png" width="85%" style="display: block; margin: auto;" />

We can show that the components $Y_j$ and $Y_k$ are:

- **asymptotically dependent** only if $\alpha_{k|j}=1$, $\beta_{k|j}=0$
- **asymptotically independent** if $-1<\alpha_{k|j}<1$
    - positive extremal dependence if $\alpha_{k|j}>0$
    - negative extremal dependence if $\alpha_{k|j}<0$
    - near independence if $\alpha_{k|j}=\beta_{k|j}=0$.
Negative values of $\beta$ ($b$ in the plots) imply that all if the conditional quantiles of $Y_k$ converge to the same value as $Y_j$. This is often implausible.

::: yourturn

Are the estimates of $\alpha_{k|j}$ and $\beta_{j|j}$ in line with your expectations? Justify your answer.

:::

## Functionalities of `mex` and diagnostic plots

The output of `mex` or `mexDependence` is an object of class `mex`, for which many `S3` methods are available (including `print` and `summary`). By default the `plot` (or `ggplot`) method produce diagnostic plots of

- standardized residuals $\widehat{Z}_k(Y_j)$ against $\widehat{F}_j(Y_j)$ with lowess curve
- absolute value of standardized residuals against $\widehat{F}_j(Y_j)$
- plot of $Y_k$ against $Y_j$ with fitted quantiles

If there is a trend, this contradicts somewhat the assumption that the residuals $Z$ are independent of the value of $Y_j$, and perhaps suggest that the threshold used for the dependence is too low.


```r
ggplot(condModFit)
```

<div class="figure" style="text-align: center">
<img src="texmex_files/figure-html/diagMexPlots-1.png" alt="Regression diagnostic plots for the fitted conditional extremes model." width="85%" />
<p class="caption">(\#fig:diagMexPlots)Regression diagnostic plots for the fitted conditional extremes model.</p>
</div>


## Uncertainty quantification 

:::keyidea

* Uncertainty quantification is obtained using a semiparametric bootstrap scheme.
* Computationally expensive, replicate every step of the analysis with bootstrap data.

:::

The drawback of the "bricolage" approach for model fitting is that we do not have readily available uncertainty quantification. Heffernan and Tawn address this by devising a semiparametric bootstrap scheme.
It consists of the following steps:

- obtain nonparametric bootstrap sample (with replacement) from $\{\boldsymbol{Y}_i, \ldots, \boldsymbol{Y}_n\}$
- obtain $D$ samples of size $n$ from the standard Laplace margins $\boldsymbol{Z}^{(b)}$.
- reorder Laplace observations of each margin so that their rank match that of the nonparametric bootstrap.
- transform observations to the data scale using $\widehat{F}_j^{-1}(\cdot)$ $(j=1, \ldots, D)$

Thus, we have a new dataset from which to estimate the dependence. Rince and repeat every step including

- empirical distribution estimation
- generalized Pareto above threshold
- multivariate model with coefficients

The `bootmex` package takes the output of `mex` or `mexDependence` and produces `R` replicates.


```r
bootCondModFit <- bootmex(condModFit)
```

So far, we have not assessed the choice of threshold for the procedure. We can produce a threshold stability plot by fitting the model over a range of values of `dqu` and look at the resulting estimates. This, coupled with some bootstrap replicates, is useful to make sure the optimization algorithm did converge and that the threshold is high enough for the approximation to hold.


```r
threshold_stab_multi <-
  mexRangeFit(
    x = marg, 
    quantiles = seq(0.90, 0.98, by = 0.02),
    which = "precip", #margin to condition on
    R = 10L, # number of bootstrap samples
    trace = Inf
  )
ggplot(threshold_stab_multi)
```

<div class="figure" style="text-align: center">
<img src="texmex_files/figure-html/thresholdStabPlots-1.png" alt="Dependence threshold stability plots with 10 bootstrap replicates for the conditional extreme value model fitted above the 0.9 quantile of cumulated daily precipitation." width="85%" />
<p class="caption">(\#fig:thresholdStabPlots)Dependence threshold stability plots with 10 bootstrap replicates for the conditional extreme value model fitted above the 0.9 quantile of cumulated daily precipitation.</p>
</div>

We can use the resulting object to produce scatterplots of pairs of parameter estimates for each margin and get bootstrap confidence intervals or standard errors for the parameters.


```r
# ggplot method seems broken...
par(mfrow = c(1,2), bty = "l")
plot(bootCondModFit, plot = "dependence")
```

<div class="figure" style="text-align: center">
<img src="texmex_files/figure-html/bootstrapPairsPlot-1.png" alt="Scatterplot of bootstrap parameter estimates for the coefficients of the conditional extreme value model for maximum daily temperature (left) and minimum daily temperature (right) conditional on cumulated daily precipitation. The maximum likelihood estimate for the original data is indicated with the at-sign (red)." width="85%" />
<p class="caption">(\#fig:bootstrapPairsPlot)Scatterplot of bootstrap parameter estimates for the coefficients of the conditional extreme value model for maximum daily temperature (left) and minimum daily temperature (right) conditional on cumulated daily precipitation. The maximum likelihood estimate for the original data is indicated with the at-sign (red).</p>
</div>



# Estimating the probability of rare events



:::keyidea

* Estimation of risk measures via Monte Carlo methods: simulate new replicates from the fitted model and compute number of points
* For general risk regions where one or more component can be extreme, we proceed by fitting multiple conditional models
* Entirely empirical approach

:::

The whole point of estimating multivariate model is to estimate the probability of rare events. In general, we can estimate the probability of falling in a risk region $\mathcal{A} \subset\{Y_j > u_j\}$ via Monte Carlo simulations. In `texmex`, this is achieved with the `predict` method, which uses the model for simulation.

Under the hood, the function forms $n$ vectors of residuals 
\[
\widehat{\boldsymbol{z}}_{i|j} = y_{ij}^{-\widehat{\boldsymbol{\beta}}_{|j}}\left(\boldsymbol{y}_{i,-j} - \widehat{\boldsymbol{\alpha}}_{|j}y_{ij}\right), \quad (i=1, \ldots, n).
\]

Then, choosing a threshold $v > u_j$, we can

1. Simulate $Y_j \sim \mathsf{Exp}(1) + v$.
2. Sample $\boldsymbol{z}_{|j}$ uniformly from the empirical distribution $\{\widehat{\boldsymbol{z}}_{i|j}\}_{i=1}^n$.
3. Set $\boldsymbol{Y}_{-j}= \widehat{\boldsymbol{\alpha}}Y_j + Y_j^{\widehat{\boldsymbol{\beta}}}\boldsymbol{z}_{|j}$.
4. Back-transform observations to original scale.


The `predict` method for objects of class `mex` and `bootmex` performs `nsim=1000` replications by default above `pqu=0.99`. We may choose the dependence threshold over which to predict the other variables. Note that the bootstrap estimates contain in addition to the quantiles estimated standard error for the conditional mean. For the Fort Collins data, they appear to give much narrower quantile ranges.

The `predict` method includes an option `smoothZdistribution` to perform kernel smoothing for each marginal to remove artefacts. The levels of the conditional quantiles can be changed via the argument `probs`.


```r
pred_boot <- predict(
	bootCondModFit, 
	pqu = 0.9,	
	trace = Inf)
# for the `bootmex` object, produces replications 
# for each bootstrap sample
pred_mex <- predict(condModFit, pqu = 0.9)
# `print` will output conditional mean
# `summary` gives conditional mean and quantiles
summary(pred_mex)
#> predict.mex(object = condModFit, pqu = 0.9)
#> 
#> Conditioned on precip being above its 90th percentile.
#> 
#> 
#> Conditional Mean and Quantiles:
#> 
#>      precip|precip>Q90 maxTemp|precip>Q90 minTemp|precip>Q90
#> mean              8.99              14.10               3.79
#> 5%                2.42              -2.25             -11.70
#> 50%               5.98              14.40               5.56
#> 95%              27.10              29.50              15.80
#> 
#> Conditional probability of threshold exceedance:
#> 
#>  P(precip>2.16|precip>Q90) P(maxTemp>30|precip>Q90)
#>                          1                    0.038
#>  P(minTemp>12.7778|precip>Q90)
#>                          0.158
summary(pred_boot)
#> predict.mex(object = bootCondModFit, pqu = 0.9, trace = Inf)
#> 
#> Results from 100 bootstrap runs.
#> 
#> Conditioned on precip being above its 90th percentile.
#> 
#> 
#> Conditional Mean and Quantiles:
#> 
#>      precip|precip>Q90 maxTemp|precip>Q90 minTemp|precip>Q90
#> mean             9.060             13.900              3.480
#> se               0.397              0.412              0.361
#> 5%               8.440             13.200              2.900
#> 50%              9.040             13.900              3.490
#> 95%              9.790             14.500              4.190
#> 
#> Conditional probability of threshold exceedance:
#> 
#>  P(precip>2.16|precip>Q90) P(maxTemp>30|precip>Q90)
#>                          1                   0.0335
#>  P(minTemp>12.7778|precip>Q90)
#>                          0.148
```

We can produce plots of `predict.mex` objects using `ggplot`. The resulting scatterplots are slightly confusing:

- the orange full line gives the line $t_j(Y_j) = t_k(Y_k)$; since the data are plotted on the scale $(Y_j, Y_k)$ of the data, the line is curved.
- the vertical dashed line represent the dependence threshold
- grey points are original observations
- blue diamonds are new simulated observations from the model for which the simulated conditioning variable $Y_j > Y_k$; orange circles are simulated points for which $Y_k > Y_j$.


<div class="figure" style="text-align: center">
<img src="texmex_files/figure-html/predictPlots-1.png" alt="Simulations from the fitted conditional model above the 0.9 quantile (blue or orange) with original data (grey). The orange line indicates equal quantiles for the pair." width="85%" />
<p class="caption">(\#fig:predictPlots)Simulations from the fitted conditional model above the 0.9 quantile (blue or orange) with original data (grey). The orange line indicates equal quantiles for the pair.</p>
</div>

Thus, we can see that the simulation scheme produces coherent data in this case. The simulations are stored in `pred_mex$data`, a list which contains the original data (`real`), simulated observations on the data scale (`simulated`) and on the Laplace (`transformed`) scale.

We can access these data and compute empirical estimates of probabilities for various sets of interest.

::: yourturn

Estimate the probability that the maximum daily temperature is below 30 degrees Celcius if the amount of rainfall is larger than 25 mm.

:::

## Self-consistency

So far, our model is only valid in the region $Y_j > u_j$, but more often than not we will be interested in more general events of the form $\{Y_j>u_j, Y_k>u_k\}$. Depending on the conditioning variable, we could express this in two ways:
\begin{align*}
\Pr(Y_k > u_k, Y_j > u_j) &= \Pr(Y_k > u_k \mid Y_j > u_j)\Pr(Y_j > u_j)
\\& = \Pr(Y_j > u_j \mid Y_k > u_k)\Pr(Y_k > u_k)
\end{align*}
and if we choose a different conditioning variable, we will obtain different answers. This is one of the major drawback of the approach: it does not readily yield a genuine multivariate model and it is not self-consistent (meaning that the answer to the above question is the same regardless of the conditioning variable if $t_j(u_j) = t_k(u_k)$). For this to be the case, one would need to enforce additional constraints for $a_{j|k}=a_{k|j}$, etc.


## Glue together conditional models

@Heffernan:2004 suggest splitting the risk region $\mathcal{A}$ into disjoint sets for exceedances, choosing the conditioning variable with the largest component on the Laplace scale. If we view the model as a mixture with adequate weights, this defines a valid multivariate distribution [cf. discussion of I. Papastathopoulos in @Engelke/Hitz:2020], but the model fitting procedure does not factor this into account: it fits the model parameters $\boldsymbol{\alpha}_{|j}$ and $\boldsymbol{\beta}_{|j}$ with all data $\{\boldsymbol{Y}_i: Y_j>u_j\}$, not just that for which $\{\boldsymbol{Y}_i: Y_j>u_j, t_j(Y_j) > t_k(Y_k), k \neq j\}$.


The function `mexAll` fits all models conditional models together. One can then use `mexMonteCarlo` to generate points from each of these models, where in each case the $i$th bootstrap observation is replaced by a sample from the $j$th conditional extremes model, that is that for which $j$ is the largest standardized component occurs, i.e. $t_j(Y_{ij}) > t_k(Y_{ik})$ for $k \neq j$.
