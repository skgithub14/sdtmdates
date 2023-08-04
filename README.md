
<!-- README.md is generated from README.Rmd. Please edit that file -->

# sdtmdates

<!-- badges: start -->

[![R-CMD-check](https://github.com/skgithub14/sdtmdates/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/skgithub14/sdtmdates/actions/workflows/R-CMD-check.yaml)
[![Codecov test
coverage](https://codecov.io/gh/skgithub14/sdtmdates/branch/main/graph/badge.svg)](https://app.codecov.io/gh/skgithub14/sdtmdates?branch=main)
<!-- badges: end -->

The goal of {sdtmdates} is to provide a set of tools for statistical
programmers to transform raw electronic data cut (EDC) dates into ISO
8601 formatted dates for Study Data Tabulation Model (SDTM) data sets.
The tools include utility functions to reshaping, trimming, and imputing
date values.

## Installation

You can install the development version of {sdtmdates} from
[GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("skgithub14/sdtmdates")
```

## Example

In this example, we start with a data frame with two columns, one with
full dates and one with partial dates. The goal is to consolidate these
dates into one ISO 8601 formatted date column.

``` r
library(sdtmdates)
library(dplyr)
library(knitr)

raw_dates <- data.frame(
  raw_full = c(
    rep(NA, 8),
    "02/05/2017",
    "02-05-2017"
  ),
  raw_partial = c(
    "UN-UNK-UNKN", 
    "UN/UNK/UNKN",
    "UN UNK UNKN",
    "UN-UNK-2017",
    "UN-Feb-2017",
    "05-FEB-2017",
    "05-UNK-2017",
    "05-Feb-UNKN",
    rep(NA, 2)
  )
)
kable(raw_dates)
```

| raw_full   | raw_partial |
|:-----------|:------------|
| NA         | UN-UNK-UNKN |
| NA         | UN/UNK/UNKN |
| NA         | UN UNK UNKN |
| NA         | UN-UNK-2017 |
| NA         | UN-Feb-2017 |
| NA         | 05-FEB-2017 |
| NA         | 05-UNK-2017 |
| NA         | 05-Feb-UNKN |
| 02/05/2017 | NA          |
| 02-05-2017 | NA          |

First, we will re-arrange the partial dates into the same format as the
full dates using `reshape_pdates()`. That will let us combine the full
and partial dates into one column with a MM/DD/YYYY format. Then, using
`reshape_adates()`, we will converted the dates to the YYYY-MM-DD
format.

``` r
working_dates <- raw_dates %>%
  mutate(
    partial = reshape_pdates(raw_partial),
    all = coalesce(raw_full, partial),
    all = reshape_adates(all)
  )
kable(working_dates)
```

| raw_full   | raw_partial | partial    | all        |
|:-----------|:------------|:-----------|:-----------|
| NA         | UN-UNK-UNKN | UN/UN/UNKN | UNKN-UN-UN |
| NA         | UN/UNK/UNKN | UN/UN/UNKN | UNKN-UN-UN |
| NA         | UN UNK UNKN | UN/UN/UNKN | UNKN-UN-UN |
| NA         | UN-UNK-2017 | UN/UN/2017 | 2017-UN-UN |
| NA         | UN-Feb-2017 | 02/UN/2017 | 2017-02-UN |
| NA         | 05-FEB-2017 | 02/05/2017 | 2017-02-05 |
| NA         | 05-UNK-2017 | UN/05/2017 | 2017-UN-05 |
| NA         | 05-Feb-UNKN | 02/05/UNKN | UNKN-02-05 |
| 02/05/2017 | NA          | NA         | 2017-02-05 |
| 02-05-2017 | NA          | NA         | 2017-02-05 |

For situations where missing date elements should be removed, use the
`trim_dates()` function.

``` r
trimmed_dates <-  mutate(working_dates, trimmed = trim_dates(all))
kable(trimmed_dates)
```

| raw_full   | raw_partial | partial    | all        | trimmed    |
|:-----------|:------------|:-----------|:-----------|:-----------|
| NA         | UN-UNK-UNKN | UN/UN/UNKN | UNKN-UN-UN | NA         |
| NA         | UN/UNK/UNKN | UN/UN/UNKN | UNKN-UN-UN | NA         |
| NA         | UN UNK UNKN | UN/UN/UNKN | UNKN-UN-UN | NA         |
| NA         | UN-UNK-2017 | UN/UN/2017 | 2017-UN-UN | 2017       |
| NA         | UN-Feb-2017 | 02/UN/2017 | 2017-02-UN | 2017-02    |
| NA         | 05-FEB-2017 | 02/05/2017 | 2017-02-05 | 2017-02-05 |
| NA         | 05-UNK-2017 | UN/05/2017 | 2017-UN-05 | 2017       |
| NA         | 05-Feb-UNKN | 02/05/UNKN | UNKN-02-05 | NA         |
| 02/05/2017 | NA          | NA         | 2017-02-05 | 2017-02-05 |
| 02-05-2017 | NA          | NA         | 2017-02-05 | 2017-02-05 |

If imputed dates are needed, use the `impute_pdates()` function. Both
start and end dates can be imputed using standard imputation rules.

``` r
imputed_dates <- working_dates %>%
  mutate(
    start = impute_pdates(all, ptype = "start"),
    end = impute_pdates(all, ptype = "end")
  )
kable(imputed_dates)
```

| raw_full   | raw_partial | partial    | all        | start      | end        |
|:-----------|:------------|:-----------|:-----------|:-----------|:-----------|
| NA         | UN-UNK-UNKN | UN/UN/UNKN | UNKN-UN-UN | NA         | NA         |
| NA         | UN/UNK/UNKN | UN/UN/UNKN | UNKN-UN-UN | NA         | NA         |
| NA         | UN UNK UNKN | UN/UN/UNKN | UNKN-UN-UN | NA         | NA         |
| NA         | UN-UNK-2017 | UN/UN/2017 | 2017-UN-UN | 2017-01-01 | 2017-12-31 |
| NA         | UN-Feb-2017 | 02/UN/2017 | 2017-02-UN | 2017-02-01 | 2017-02-28 |
| NA         | 05-FEB-2017 | 02/05/2017 | 2017-02-05 | 2017-02-05 | 2017-02-05 |
| NA         | 05-UNK-2017 | UN/05/2017 | 2017-UN-05 | 2017-01-05 | 2017-12-05 |
| NA         | 05-Feb-UNKN | 02/05/UNKN | UNKN-02-05 | NA         | NA         |
| 02/05/2017 | NA          | NA         | 2017-02-05 | 2017-02-05 | 2017-02-05 |
| 02-05-2017 | NA          | NA         | 2017-02-05 | 2017-02-05 | 2017-02-05 |
