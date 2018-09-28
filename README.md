<!-- README.md is generated from README.Rmd. Please edit that file -->
deflateBR
=========

[![Travis-CI Build
Status](https://travis-ci.org/meirelesff/deflateBR.svg?branch=master)](https://travis-ci.org/meirelesff/deflateBR)
[![AppVeyor Build
Status](https://ci.appveyor.com/api/projects/status/github/meirelesff/deflateBR?branch=master&svg=true)](https://ci.appveyor.com/project/meirelesff/deflateBR)

`deflateBR` is an `R` package used to deflate nominal Brazilian Reais
using several popular price indexes.

How does it work?
-----------------

The `deflateBR`’s main function, `deflate`, requires three arguments to
work: a `numeric` vector of nominal Reais (`nominal_values`); a `Date`
vector of corresponding dates (`nominal_dates`); and a reference month
in the `MM/YYYY` format (`real_date`), used to deflate the values. An
example:

``` r
# Load the package
library(deflateBR)

# Deflate January 2000 reais
deflate(nominal_values = 100, nominal_dates = as.Date("2000-01-01"), real_date = "01/2018")
#> 
#> Downloading necessary data from IPEA's API
#> ...
#> [1] 310.3893
```

Behind the scenes, `deflateBR` requests data from
[IPEADATA](http://www.ipeadata.gov.br/)’s API on one these five
Brazilian price indexes:
[IPCA](https://ww2.ibge.gov.br/english/estatistica/indicadores/precos/inpc_ipca/defaultinpc.shtm)
and
[INPC](https://ww2.ibge.gov.br/english/estatistica/indicadores/precos/inpc_ipca/defaultinpc.shtm),
maintained by [IBGE](https://ww2.ibge.gov.br/home/); and
[IGP-M](http://portalibre.fgv.br/main.jsp?lumChannelId=402880811D8E34B9011D92B6160B0D7D),
[IGP-DI](http://portalibre.fgv.br/main.jsp?lumChannelId=402880811D8E34B9011D92B6160B0D7D),
and
[IPC](http://portalibre.fgv.br/main.jsp?lumChannelId=402880811D8E34B9011D92B7350710C7)
maintained by
[FGV/IBRE](http://portalibre.fgv.br/main.jsp?lumChannelId=402880811D8E2C4C011D8E33F5700158).
To select one of the available price indexes, just provide one of the
following options to the `index =` argument: `ipca`, `igpm`, `igpdi`,
`ipc`, and `inpc`. In the following, the INPC index is used.

``` r
# Deflate January 2000 reais using the FGV/IBRE's INCP price index
deflate(100, as.Date("2000-01-01"), "01/2018", index = "inpc")
#> 
#> Downloading necessary data from IPEA's API
#> ...
#> [1] 318.1845
```

With the same syntax, a vector of nominal Reais can also be deflated,
what is useful while working with `data.frames`:

``` r
# Create some data
df <- tibble::tibble(reais = seq(101, 200),
                     dates = seq.Date(from = as.Date("2001-01-01"), by = "month", length.out = 100)
                     )

# Reference date to deflate the nominal values
reference <- "01/2018"

# Deflate using IGP-DI
head(deflate(df$reais, df$dates, reference, "igpdi"))
#> 
#> Downloading necessary data from IPEA's API
#> ...
#> [1] 341.0412 342.7393 344.9315 345.5051 344.9379 346.6979
```

### Working with the tidyverse

For `tidyverse` users, the `deflate` function can be easily used inside
code chains:

``` r
library(tidyverse)

df %>%
  mutate(deflated_reais = deflate(reais, dates, reference, "ipca"))
#> 
#> Downloading necessary data from IPEA's API
#> ...
#> # A tibble: 100 x 3
#>    reais dates      deflated_reais
#>    <int> <date>              <dbl>
#>  1   101 2001-01-01           296.
#>  2   102 2001-02-01           297.
#>  3   103 2001-03-01           299.
#>  4   104 2001-04-01           300.
#>  5   105 2001-05-01           301.
#>  6   106 2001-06-01           303.
#>  7   107 2001-07-01           304.
#>  8   108 2001-08-01           303.
#>  9   109 2001-09-01           304.
#> 10   110 2001-10-01           306.
#> # ... with 90 more rows
```

### Convenience functions

To facilitate the task of deflating nominal Reais, the `deflateBR`
package also provides five convenience functions: `ipca`, `inpc`,
`igpm`, `igpdi`, and `ipc`. Each one of these functions deflate nominal
values using their corresponding price indexes. For instance, to deflate
nominal Reais using IGP-M, one can execute the following code:

``` r
igpm(100, as.Date("2000-01-01"), "01/2018")
```

Or, using the IPCA index:

``` r
ipca(100, as.Date("2000-01-01"), "01/2018")
```

In addition, the `deflateBR` package contains a function called
`inflation` that calculates the inflation rate between two dates
quickly. Providing initial and end dates in the MM/YYYY format, plus one
price index, the function returns the inflation rate in percent:

``` r
# Inflation rate between January and December of 2017
inflation("01/2017", "12/2017", "ipca")
#> 
#> Downloading necessary data from IPEA's API
#> ...
#> [1] 2.947421
```

Installing
----------

Install the latest released version from CRAN with:

``` r
install.packages("deflateBR")
```

To install the development version of the package, use:

``` r
if (!require("devtools")) install.packages("devtools")
devtools::install_github("meirelesff/deflateBR")
```

Methodology
-----------

Following standard practice, seconded by the [Brazilian Central
Bank](https://www3.bcb.gov.br/CALCIDADAO/publico/metodologiaCorrigirIndice.do?method=metodologiaCorrigirIndice),
the `deflateBR` adjusts for inflation by multiplying nominal Reais by
the ratio between the original and the reference price indexes. For
example, to adjust 100 reais of January 2018, with IPCA index of
4916.46, to August 2018, with IPCA of 5056.56 in the previous month, we
first calculate the ratio between the two indexes (e.g., 5056.56 /
4916.46 = 1.028496) and then multiply this value by 100 (e.g., 102.84
adjusted Reais). The `deflate` function gives exactly the same result:

``` r
deflate(100, as.Date("2018-01-01"), "08/2018", "ipca")
#> 
#> Downloading necessary data from IPEA's API
#> ...
#> [1] 102.8496
```

Citation
--------

To cite `deflateBR` in publications, please use:

``` r
citation('deflateBR')
```

Author
------

[Fernando Meireles](http://fmeireles.com)
