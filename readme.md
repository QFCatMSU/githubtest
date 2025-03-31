# Parametric survival modeling with censoring via the Weibull
distribution
Chris Cahill

## Description of files

‘weibull_play.R’ shows some simple ways to simulate and estimate Weibull
data

‘compare.R’ shows relationships between accelerated failure time and log
hazard parameterizations of the Weibull model

## Introduction

This document derives and implements the Weibull log-likelihood with
optional right censoring. We walk through the probability density
function (PDF), survival function, cumulative hazard, and the full
log-likelihood step-by-step, with algebraic derivations and
interpretation. We also demonstrate this with simulated data. Much of
this document comes from the Weibull distribution (see Wikipedia) and
the first 100 pages of Paul Allison’s Survival Analysis using SAS book.

## 1. Weibull Probability Density Function (PDF)

The Weibull distribution can parameterized by shape
![k](https://latex.codecogs.com/svg.latex?k "k") and scale
![\lambda](https://latex.codecogs.com/svg.latex?%5Clambda "\lambda"):

![f(t) = \frac{k}{\lambda} \left(\frac{t}{\lambda}\right)^{k - 1} e^{-\left(\frac{t}{\lambda}\right)^k}](https://latex.codecogs.com/svg.latex?f%28t%29%20%3D%20%5Cfrac%7Bk%7D%7B%5Clambda%7D%20%5Cleft%28%5Cfrac%7Bt%7D%7B%5Clambda%7D%5Cright%29%5E%7Bk%20-%201%7D%20e%5E%7B-%5Cleft%28%5Cfrac%7Bt%7D%7B%5Clambda%7D%5Cright%29%5Ek%7D "f(t) = \frac{k}{\lambda} \left(\frac{t}{\lambda}\right)^{k - 1} e^{-\left(\frac{t}{\lambda}\right)^k}")

Both ![k](https://latex.codecogs.com/svg.latex?k "k") and
![\lambda](https://latex.codecogs.com/svg.latex?%5Clambda "\lambda")
must be \> 0 for the Weibull distribution. In terms of interpretation,
![k](https://latex.codecogs.com/svg.latex?k "k") can be thought of as
modulating the risk or hazard of an event happening through time while
![\lambda](https://latex.codecogs.com/svg.latex?%5Clambda "\lambda")
modulates the horizontal stretch of the distribution.

For example, if ![k](https://latex.codecogs.com/svg.latex?k "k") \< 1.0,
some fish leave quickly while others linger. If
![k](https://latex.codecogs.com/svg.latex?k "k") = 1.0, it means there
is a constant hazard through time (i.e., this is an exponential model).
If ![k](https://latex.codecogs.com/svg.latex?k "k") \> 1.0, the hazard
increases through time so fish are more likely to leave as time goes on.
In this case, ![k](https://latex.codecogs.com/svg.latex?k "k") can be
thought of as describing the risk profile of fish.

For
![\lambda](https://latex.codecogs.com/svg.latex?%5Clambda "\lambda"),
higher values mean fish tend to move or transition later, all other
things being equal. It describes the timing of movement events.

We simulate a bunch of distributions below to visualize this, but first
we’ll walk through some important math involving the Weibull and derive
a loglikelihood for censored data.

## 2. Deriving the Cumulative Distribution Function (CDF)

Integrating the PDF provides the CDF

![F(t) = 1 - \exp\left(-\left(\frac{t}{\lambda}\right)^k\right)](https://latex.codecogs.com/svg.latex?F%28t%29%20%3D%201%20-%20%5Cexp%5Cleft%28-%5Cleft%28%5Cfrac%7Bt%7D%7B%5Clambda%7D%5Cright%29%5Ek%5Cright%29 "F(t) = 1 - \exp\left(-\left(\frac{t}{\lambda}\right)^k\right)")

## 3. Survival Function

The survival function is then the complement of the CDF:

![S(t) = 1 - F(t) = \exp\left(-\left(\frac{t}{\lambda}\right)^k\right)](https://latex.codecogs.com/svg.latex?S%28t%29%20%3D%201%20-%20F%28t%29%20%3D%20%5Cexp%5Cleft%28-%5Cleft%28%5Cfrac%7Bt%7D%7B%5Clambda%7D%5Cright%29%5Ek%5Cright%29 "S(t) = 1 - F(t) = \exp\left(-\left(\frac{t}{\lambda}\right)^k\right)")

## 4. Hazard Function

Defined as:

![h(t) = \frac{f(t)}{S(t)} = \frac{k}{\lambda} \left(\frac{t}{\lambda}\right)^{k - 1}](https://latex.codecogs.com/svg.latex?h%28t%29%20%3D%20%5Cfrac%7Bf%28t%29%7D%7BS%28t%29%7D%20%3D%20%5Cfrac%7Bk%7D%7B%5Clambda%7D%20%5Cleft%28%5Cfrac%7Bt%7D%7B%5Clambda%7D%5Cright%29%5E%7Bk%20-%201%7D "h(t) = \frac{f(t)}{S(t)} = \frac{k}{\lambda} \left(\frac{t}{\lambda}\right)^{k - 1}")

## 5. Log-Hazard Formulation

Taking logs:

![\log h(t) = \log k - \log \lambda + (k - 1) \log t](https://latex.codecogs.com/svg.latex?%5Clog%20h%28t%29%20%3D%20%5Clog%20k%20-%20%5Clog%20%5Clambda%20%2B%20%28k%20-%201%29%20%5Clog%20t "\log h(t) = \log k - \log \lambda + (k - 1) \log t")

This is equivalent to:

![\log h(t) = a + b \log t](https://latex.codecogs.com/svg.latex?%5Clog%20h%28t%29%20%3D%20a%20%2B%20b%20%5Clog%20t "\log h(t) = a + b \log t")

Where:

- ![(a = \log k - k \log \lambda)](https://latex.codecogs.com/svg.latex?%28a%20%3D%20%5Clog%20k%20-%20k%20%5Clog%20%5Clambda%29 "(a = \log k - k \log \lambda)")
- ![(b = k - 1)](https://latex.codecogs.com/svg.latex?%28b%20%3D%20k%20-%201%29 "(b = k - 1)")

## 6. Cumulative Hazard Function

We need the cumulative hazard function H(t) because it captures the
total risk of failure accumulated up to time t. This is important even
for censored observations, since knowing that someone survived up to a
certain point still provides useful information.

The cumulative hazard can be derived by taking the integral of the
hazard function and in our case is:

![H(t) = -\log S(t) = \left(\frac{t}{\lambda}\right)^k](https://latex.codecogs.com/svg.latex?H%28t%29%20%3D%20-%5Clog%20S%28t%29%20%3D%20%5Cleft%28%5Cfrac%7Bt%7D%7B%5Clambda%7D%5Cright%29%5Ek "H(t) = -\log S(t) = \left(\frac{t}{\lambda}\right)^k")

In log-hazard form:

![H(t) = \frac{\exp(a) \cdot t^{b+1}}{b+1}](https://latex.codecogs.com/svg.latex?H%28t%29%20%3D%20%5Cfrac%7B%5Cexp%28a%29%20%5Ccdot%20t%5E%7Bb%2B1%7D%7D%7Bb%2B1%7D "H(t) = \frac{\exp(a) \cdot t^{b+1}}{b+1}")

## 7. Log-Likelihood with Censoring

Let
![(d_i = 1)](https://latex.codecogs.com/svg.latex?%28d_i%20%3D%201%29 "(d_i = 1)")
if the event is observed, and 0 if censored. Then the likelihood for
observation (i) is:

![L_i = \[f(t_i)\]^{d_i} \[S(t_i)\]^{1 - d_i} = \[h(t_i) S(t_i)\]^{d_i} \[S(t_i)\]^{1 - d_i} = \[h(t_i)\]^{d_i} S(t_i)](https://latex.codecogs.com/svg.latex?L_i%20%3D%20%5Bf%28t_i%29%5D%5E%7Bd_i%7D%20%5BS%28t_i%29%5D%5E%7B1%20-%20d_i%7D%20%3D%20%5Bh%28t_i%29%20S%28t_i%29%5D%5E%7Bd_i%7D%20%5BS%28t_i%29%5D%5E%7B1%20-%20d_i%7D%20%3D%20%5Bh%28t_i%29%5D%5E%7Bd_i%7D%20S%28t_i%29 "L_i = [f(t_i)]^{d_i} [S(t_i)]^{1 - d_i} = [h(t_i) S(t_i)]^{d_i} [S(t_i)]^{1 - d_i} = [h(t_i)]^{d_i} S(t_i)")

Taking logs:

![\log L = \sum_i \left\[ d_i \log h(t_i) + (1 - d_i) \log S(t_i) \right\] = \sum_i d_i \log h(t_i) - H(t_i)](https://latex.codecogs.com/svg.latex?%5Clog%20L%20%3D%20%5Csum_i%20%5Cleft%5B%20d_i%20%5Clog%20h%28t_i%29%20%2B%20%281%20-%20d_i%29%20%5Clog%20S%28t_i%29%20%5Cright%5D%20%3D%20%5Csum_i%20d_i%20%5Clog%20h%28t_i%29%20-%20H%28t_i%29 "\log L = \sum_i \left[ d_i \log h(t_i) + (1 - d_i) \log S(t_i) \right] = \sum_i d_i \log h(t_i) - H(t_i)")

### Interpretation:

- ![\log h(t_i)](https://latex.codecogs.com/svg.latex?%5Clog%20h%28t_i%29 "\log h(t_i)"):
  Only contributes when the event is observed
  ![d_i = 1](https://latex.codecogs.com/svg.latex?d_i%20%3D%201 "d_i = 1").
  It reflects the **instantaneous risk**.
- ![-H(t_i)](https://latex.codecogs.com/svg.latex?-H%28t_i%29 "-H(t_i)"):
  Always contributes. It reflects the **probability of surviving** up to
  time ![t_i](https://latex.codecogs.com/svg.latex?t_i "t_i").

Together, they form the complete log-likelihood for censored survival
data.

## 9. Simulating Weibull Distributions

``` r
# --- panel 1: varying shape, fixed scale ---
set.seed(123)
k_vals <- c(0.5, 1, 1.5, 2)
lambda_vals <- rep(1, length(k_vals))
colors <- c("blue", "dodgerblue3", "purple", "darkorchid")
times <- seq(0.01, 5, length.out = 200)

par(mfrow = c(2, 2), mar = c(4, 4, 2, 1))

# PDF
plot(NULL, xlim = c(0, 5), ylim = c(0, 2), xlab = "t", ylab = "f(t)", 
     main = "PDF (varying k)")
for (i in seq_along(k_vals)) {
  lines(times, dweibull(times, shape = k_vals[i], scale = lambda_vals[i]), 
        col = colors[i], lwd = 2)
}
legend("topright", legend = paste0("k=", k_vals), col = colors, lwd = 2)

# Hazard
plot(NULL, xlim = c(0, 5), ylim = c(0, 8), xlab = "t", ylab = "h(t)", 
     main = "Hazard (varying k)")
for (i in seq_along(k_vals)) {
  h <- (k_vals[i] / lambda_vals[i]) * (times / lambda_vals[i])^(k_vals[i] - 1)
  lines(times, h, col = colors[i], lwd = 2)
}

# Survival
plot(NULL, xlim = c(0, 5), ylim = c(0, 1), xlab = "t", ylab = "S(t)", 
     main = "Survival (varying k)")
for (i in seq_along(k_vals)) {
  S <- exp(-(times / lambda_vals[i])^k_vals[i])
  lines(times, S, col = colors[i], lwd = 2)
}

# CDF
plot(NULL, xlim = c(0, 5), ylim = c(0, 1), xlab = "t", ylab = "F(t)", 
     main = "CDF (varying k)")
for (i in seq_along(k_vals)) {
  F <- 1 - exp(-(times / lambda_vals[i])^k_vals[i])
  lines(times, F, col = colors[i], lwd = 2)
}
```

![](readme_files/figure-commonmark/unnamed-chunk-1-1.png)

``` r
# --- panel 2: varying scale, fixed shape ---
lambda_vals2 <- c(0.5, 1, 2, 3)
k_vals2 <- rep(1.5, length(lambda_vals2))
colors2 <- c("deepskyblue4", "blue", "blueviolet", "purple")

par(mfrow = c(2, 2), mar = c(4, 4, 2, 1))

# PDF
plot(NULL, xlim = c(0, 6), ylim = c(0, 2), xlab = "t", ylab = "f(t)", 
     main = "PDF (varying lambda)")
for (i in seq_along(lambda_vals2)) {
  lines(times, dweibull(times, shape = k_vals2[i], scale = lambda_vals2[i]), 
        col = colors2[i], lwd = 2)
}
legend("topright", legend = paste0("λ=", lambda_vals2), col = colors2, lwd = 2)

# Hazard
plot(NULL, xlim = c(0, 6), ylim = c(0, 5), xlab = "t", ylab = "h(t)", 
     main = "Hazard (varying lambda)")
for (i in seq_along(lambda_vals2)) {
  h <- (k_vals2[i] / lambda_vals2[i]) * 
       (times / lambda_vals2[i])^(k_vals2[i] - 1)
  lines(times, h, col = colors2[i], lwd = 2)
}

# Survival
plot(NULL, xlim = c(0, 6), ylim = c(0, 1), xlab = "t", ylab = "S(t)", 
     main = "Survival (varying lambda)")
for (i in seq_along(lambda_vals2)) {
  S <- exp(-(times / lambda_vals2[i])^k_vals2[i])
  lines(times, S, col = colors2[i], lwd = 2)
}

# CDF
plot(NULL, xlim = c(0, 6), ylim = c(0, 1), xlab = "t", ylab = "F(t)", 
     main = "CDF (varying lambda)")
for (i in seq_along(lambda_vals2)) {
  F <- 1 - exp(-(times / lambda_vals2[i])^k_vals2[i])
  lines(times, F, col = colors2[i], lwd = 2)
}
```

![](readme_files/figure-commonmark/unnamed-chunk-1-2.png)

This simulation visualizes how the Weibull distribution behaves under
different shape values with a fixed scale. Shape
![k \< 1](https://latex.codecogs.com/svg.latex?k%20%3C%201 "k < 1")
leads to decreasing hazard,
![k = 1](https://latex.codecogs.com/svg.latex?k%20%3D%201 "k = 1") gives
constant hazard (exponential), and
![k \> 1](https://latex.codecogs.com/svg.latex?k%20%3E%201 "k > 1")
gives increasing hazard.

## 8. R Implementation (Log-Likelihood Function)

This is an implementation I built for use with e.g., `optim()`:

``` r
nll_loghaz_shape_scale <- function(par, t, d) {
  k <- exp(par[1])
  lambda <- exp(par[2])
  a <- log(k) - k * log(lambda)
  b <- k - 1
  logt <- log(t)
  eta <- a + b * logt
  h <- exp(eta)                                 # hazard function h(tᵢ)
  H <- exp(a) * t^(b + 1) / (b + 1)             # cumulative hazard H(tᵢ)
  ll <- d * log(h) - H                          # full log-likelihood
  return(-sum(ll))                              # return negative log-likelihood
}
```

This then represents an estimation problem and the various tools used
with maximum likelihood estimation can be used for inference.
