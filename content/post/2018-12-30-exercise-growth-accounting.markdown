---
title: "An Exercise in Growth Accounting"
author: "Franz X. Mohr"
date: '2018-12-30'
tags:
  - growth-accounting
  - solow-residual
  - pwt
---

The discipline of growth accounting tries to assess the relative contribution of labour, capital and technology to the economic growth of a country. This post provides a short theoretical introduction to the concept of growth accounting and uses Penn World Table data to calculate total factor productivity (TFP) growth rates for a series of countries using the simple Solow-method.

<!--more-->

## Theory

Based on the growth model of Solow (1957) the discipline of growth accounting tries to assess the relative contribution of labour, capital and technology to the economic growth of a country. It departs from the idea that the economy can be described by a single production function of the form
`$$Y= A K^{\alpha}L^{1-\alpha,}$$`
which becomes `\(y=Ak^{\alpha}\)` in per-capita terms, where `\(y = Y / L\)` is output per capita, `\(k = K / L\)` is capital per capita and `\(A\)` is total factor productivity (TFP). Taking logs and differentiating with respect to time gives an expression of the economy's growth rate `\(g = \frac{\dot{y}}{y}\)`, which depends on TFP growth `\(\frac{\dot{A}}{A}\)` and capital growth `\(\frac{\dot{k}}{k}\)` over time so that
`$$g = \frac{\dot{A}}{A} + \alpha \frac{\dot{k}}{k}.$$`

Since output, capital and labour can be observed, but `\(A\)` cannot, the latter has to be calculated in some way. The most basic approach to do so is the so-called *residual method*, which assumes that `\(\alpha\)` is equal to the share of capital income in national income.[^alpha] Once this value is obtained the TFP growth rate can be obtained by

`$$\frac{\dot{A}}{A} = g - \alpha \frac{\dot{k}}{k}.$$`

Note that since TFP growth is the residual value after the contribution of capital growth was subtracted from output growth, TFP growth is also called the *Solow residual*.

## Application in R

Applying this method in R is straightforward. A main source of data for growth-related analyses is <a href="https://www.rug.nl/ggdc/productivity/pwt/" target="_blank">Penn World Table</a>, which is also available on CRAN.


```r
# Load package
library(pwt9) # install.packages("pwt9")

# Load data
data("pwt9.0")
```

Next, we select some countries in the sample, where observations from 1960 to 2000 are available. This period is chosen, because this is also the period used for the growth accounting example in Aghion and Howitt (2009).


```r
library(dplyr)

data <- pwt9.0 %>%
  filter(country %in% c("Australia", "Austria", "Belgium",
                        "Germany", "France", "United Kingdom",
                        "United States of America")) %>%
  select(year, isocode, rgdpna, rkna, emp, labsh) %>%
  filter(year >= 1960,
         year <= 2000) %>%
  na.omit()
```

Then calculate the real output and capital per worker (in national currency), take logs and the first difference to obtain percentage changes. `\(\alpha\)` is one minus the share of labour compensation in GDP at current national prices, i.e. variable `labsh` in the dataset. You have to omit the first observation so that you have the same amount of observations as of output and capital growth.


```r
data <- data %>%
  mutate(y_pc = log(rgdpna / emp), # GDP per worker
         k_pc = log(rkna / emp), # Capital per worker
         a = 1 - labsh) %>% # Capital share
  arrange(year) %>% # Order by year
  group_by(isocode) %>% # For each country calculate...
  mutate(g = (y_pc - lag(y_pc)) * 100, # ...the growth rate of GDP per capita
         dk = (k_pc - lag(k_pc)) * 100, # ...the growth rate of capital per capita
         dsolow = g - a * dk) %>% # ...the Solow residual
  na.omit()

solow <- data %>%
  summarise(g = mean(g),
            tfp.growth = mean(dsolow),
            capital.deepening = mean(a * dk),
            tfp.share = mean(dsolow) / mean(g),
            capital.share = mean(a))

# Print output
solow
```

```
## # A tibble: 7 x 6
##   isocode     g tfp.growth capital.deepening tfp.share capital.share
##   <fct>   <dbl>      <dbl>             <dbl>     <dbl>         <dbl>
## 1 AUS      1.72       1.16             0.565     0.672         0.348
## 2 AUT      3.04       1.63             1.41      0.536         0.366
## 3 BEL      2.68       1.70             0.981     0.635         0.358
## 4 DEU      2.69       1.51             1.18      0.562         0.330
## 5 FRA      2.82       1.81             1.02      0.639         0.307
## 6 GBR      2.19       1.28             0.910     0.584         0.394
## 7 USA      1.81       1.26             0.545     0.698         0.370
```

The results are very similar to the numbers in table 5.1 in Aghion and Howitt (2009).


## Literature

Aghion, Philippe; Howitt, Peter (2009): The Economics of Growth: The MIT Press (MIT Press Books).

Solow, Robert M. (1957): Technical Change and the Aggregate Production Function. In The Review of Economics and Statistics 39 (3), pp. 312–320. Available online at http://www.jstor.org/stable/1926047.

[^alpha]: This is based on the assumption of perfectly competitive captial markets, where the marginal product of capital is equal to the rental price of capital.
