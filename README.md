
<!-- README.md is generated from README.Rmd. Please edit that file -->

# R/`medoutcon`

[![Travis-CI Build
Status](https://travis-ci.org/nhejazi/medoutcon.svg?branch=master)](https://travis-ci.org/nhejazi/medoutcon)
[![AppVeyor Build
Status](https://ci.appveyor.com/api/projects/status/github/nhejazi/medoutcon?branch=master&svg=true)](https://ci.appveyor.com/project/nhejazi/medoutcon)
[![Coverage
Status](https://img.shields.io/codecov/c/github/nhejazi/medoutcon/master.svg)](https://codecov.io/github/nhejazi/medoutcon?branch=master)
[![Project Status: WIP – Initial development is in progress, but there
has not yet been a stable, usable release suitable for the
public.](https://www.repostatus.org/badges/latest/wip.svg)](https://www.repostatus.org/#wip)
[![MIT
license](http://img.shields.io/badge/license-MIT-brightgreen.svg)](http://opensource.org/licenses/MIT)

> Efficient Causal Mediation Analysis Under Mediator-Outcome Confounding
> Affected by Exposure

**Authors:** [Nima Hejazi](https://nimahejazi.org), [Iván
Díaz](https://idiaz.xyz), and [Kara
Rudolph](http://biostat.jhsph.edu/~krudolph/)

-----

## What’s `medoutcon`?

The `medoutcon` R package is designed to provide facilities for
estimating a parameter that arises in a decomposition of the population
intervention causal effect into the (in)direct effects under stochastic
interventions in the setting of mediation analysis. `medoutcon` is
designed as an implementation to accompany the methodology described in
(<span class="citeproc-not-found" data-reference-id="diaz2019efficient">**???**</span>).

-----

## Installation

Install the most recent *stable release* from GitHub via
[`devtools`](https://www.rstudio.com/products/rpackages/devtools/):

``` r
devtools::install_github("nhejazi/medoutcon")
```

-----

## Example

To illustrate how `medoutcon` may be used to estimate the effect of
applying a stochastic intervention to the treatment (`A`) while keeping
the mediator(s) (`Z`) fixed, consider the following example:

``` r
library(data.table)
library(medoutcon)

# produces a simple data set based on ca causal model with mediation
make_example_data <- function(n_obs = 1000) {
  # baseline covariate -- simple, binary
  W <- replicate(2, rbinom(n_obs, 1, prob = 0.50))
  W <- as.data.table(W)
  setnames(W, c("w_1", "w_2"))

  # create treatment based on baseline W
  A <- as.numeric(rbinom(n_obs, 1, prob = rowSums(W)/3 + 0.1))

  # single mediator-outcome confounder
  z_prob <- 1 - plogis((A^2 + rowMeans(W)) / (A + rowSums(W^3) + 0.5))
  Z <- rbinom(n_obs, 1, prob = z_prob)

  # matrix of mediators
  m1_prob <- plogis((A^2 - Z) / (A + rowMeans(W) + 0.5))
  m2_prob <- 1 - plogis((A + rowSums(W)) / (Z + 0.5))
  M1 <- rbinom(n_obs, 1, prob = m1_prob)
  M2 <- rbinom(n_obs, 1, prob = m2_prob)
  M <- as.data.table(list(m_1 = M1, m_2 = M2))

  # create outcome as a linear function + white noise
  Y <- rowMeans(M) - Z + A - 0.1 * rowSums(W) +
    rnorm(n_obs, mean = 0, sd = 0.25)

  # full data structure
  data <- as.data.table(cbind(Y, M, Z, A, W))
  return(data)
}

# set seed and simulate example data
set.seed(75681)
example_data <- make_example_data()
w_names <- str_subset(colnames(example_data), "w")
m_names <- str_subset(colnames(example_data), "m")

# compute one-step estimate
os_medoutcon <- medoutcon(W = example_data[, ..w_names],
                          A = example_data$A,
                          Z = example_data$Z,
                          M = example_data[, ..m_names],
                          Y = example_data$Y,
                          estimator = "onestep",
                          estimator_args = list(cv_folds = 3))
summary(os_medoutcon)
```

For details on how to use data adaptive regression (machine learning)
techniques in the estimation of nuisance parameters, consider consulting
the vignette that accompanies the package.

-----

## Issues

If you encounter any bugs or have any specific feature requests, please
[file an issue](https://github.com/nhejazi/medoutcon/issues).

-----

## Contributions

Contributions are very welcome. Interested contributors should consult
our [contribution
guidelines](https://github.com/nhejazi/medoutcon/blob/master/CONTRIBUTING.md)
prior to submitting a pull request.

-----

## Citation

After using the `medoutcon` R package, please cite the following:

``` 
    @article{rudolph2019efficient,
      title={Efficient mediation analysis under mediator-outcome
        confounding affected by exposure},
      author={Rudolph, Kara E and D{\'\i}az, Iv{\'a}n and Hejazi, Nima S
        and {van der Laan}, Mark J},
      year={2019},
      url = {},
      doi = {},
      journal={},
      volume={},
      number={},
      pages={},
      publisher={}
    }

    @manual{hejazi2019medoutcon,
      author={Hejazi, Nima S and D{\'\i}az, Iv{\'a}n and Rudolph, Kara E},
      title = {{medoutcon}: Causal mediation analysis under
        mediator-outcome confounding in {R}},
      year  = {2019},
      url = {https://github.com/nhejazi/medoutcon},
      note = {R package version 0.0.1}
    }
```

-----

## License

© 2019 [Nima S. Hejazi](https://nimahejazi.org)

The contents of this repository are distributed under the MIT license.
See below for details:

    MIT License
    
    Copyright (c) 2019 Nima S. Hejazi
    
    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:
    
    The above copyright notice and this permission notice shall be included in all
    copies or substantial portions of the Software.
    
    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    SOFTWARE.

-----

## References
