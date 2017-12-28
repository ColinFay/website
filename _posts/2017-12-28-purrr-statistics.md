---
title: "A Crazy Little Thing Called {purrr} - Part 6 : doing statistics"
author: colin_fay
post_date: 2017-12-28
layout: single
permalink: /purrr-statistics/
categories: cat
output: jekyllthat::jekylldown
excerpt_separator: <!--more-->
---

This is gonna be the last time I make this Queen reference in 2017.

<!--more-->

I’ve been sharing some “stats with {purrr}” recipes on my [Twitter
account](https://twitter.com/_ColinFay) lately. Twitter being what it is
(a source of infinite temporary content), here’s the post gathering some
statistics tricks you can do with {purrr}.

## Looking for normality

If your data are in a data.frame, you can simply map a shapiro.test on
all the columns, keeping the one with a `p.value` \> to 0.05 (remember,
the shapiro tests for non-normality) :

``` r
library(purrr)
map(airquality, shapiro.test) %>% keep(~ .x$p.value > 0.05)
```

    ## $Wind
    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  .x[[i]]
    ## W = 0.98575, p-value = 0.1178

Of course, you can do the same on a non-tabular list:

``` r
# rdunif is from purrr
l <- list(a = rnorm(10), b = rdunif(1000, 10))
map(l, shapiro.test) %>% keep(~ .x$p.value > 0.05)
```

    ## $a
    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  .x[[i]]
    ## W = 0.9603, p-value = 0.7893

Also, `map_if` allows you to map only on numeric variables in your
data.frame:

``` r
map_if(iris, is.numeric, shapiro.test)
```

    ## $Sepal.Length
    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  .x[[i]]
    ## W = 0.97609, p-value = 0.01018
    ## 
    ## 
    ## $Sepal.Width
    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  .x[[i]]
    ## W = 0.98492, p-value = 0.1012
    ## 
    ## 
    ## $Petal.Length
    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  .x[[i]]
    ## W = 0.87627, p-value = 7.412e-10
    ## 
    ## 
    ## $Petal.Width
    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  .x[[i]]
    ## W = 0.90183, p-value = 1.68e-08
    ## 
    ## 
    ## $Species
    ##   [1] setosa     setosa     setosa     setosa     setosa     setosa    
    ##   [7] setosa     setosa     setosa     setosa     setosa     setosa    
    ##  [13] setosa     setosa     setosa     setosa     setosa     setosa    
    ##  [19] setosa     setosa     setosa     setosa     setosa     setosa    
    ##  [25] setosa     setosa     setosa     setosa     setosa     setosa    
    ##  [31] setosa     setosa     setosa     setosa     setosa     setosa    
    ##  [37] setosa     setosa     setosa     setosa     setosa     setosa    
    ##  [43] setosa     setosa     setosa     setosa     setosa     setosa    
    ##  [49] setosa     setosa     versicolor versicolor versicolor versicolor
    ##  [55] versicolor versicolor versicolor versicolor versicolor versicolor
    ##  [61] versicolor versicolor versicolor versicolor versicolor versicolor
    ##  [67] versicolor versicolor versicolor versicolor versicolor versicolor
    ##  [73] versicolor versicolor versicolor versicolor versicolor versicolor
    ##  [79] versicolor versicolor versicolor versicolor versicolor versicolor
    ##  [85] versicolor versicolor versicolor versicolor versicolor versicolor
    ##  [91] versicolor versicolor versicolor versicolor versicolor versicolor
    ##  [97] versicolor versicolor versicolor versicolor virginica  virginica 
    ## [103] virginica  virginica  virginica  virginica  virginica  virginica 
    ## [109] virginica  virginica  virginica  virginica  virginica  virginica 
    ## [115] virginica  virginica  virginica  virginica  virginica  virginica 
    ## [121] virginica  virginica  virginica  virginica  virginica  virginica 
    ## [127] virginica  virginica  virginica  virginica  virginica  virginica 
    ## [133] virginica  virginica  virginica  virginica  virginica  virginica 
    ## [139] virginica  virginica  virginica  virginica  virginica  virginica 
    ## [145] virginica  virginica  virginica  virginica  virginica  virginica 
    ## Levels: setosa versicolor virginica

## Bulk cor.test

As said on Twitter, this might be
[p.hacking](https://twitter.com/HBossier/status/943480783288889344), but
here’s how you can do a `cor.test` for all columns in a data.frame, with
a little help from `tidystringdist` and `broom` :

``` r
library(tidystringdist)
comb <- tidy_comb_all(names(iris))
knitr::kable(comb)
```

| V1           | V2           |
| :----------- | :----------- |
| Sepal.Length | Sepal.Width  |
| Sepal.Length | Petal.Length |
| Sepal.Length | Petal.Width  |
| Sepal.Length | Species      |
| Sepal.Width  | Petal.Length |
| Sepal.Width  | Petal.Width  |
| Sepal.Width  | Species      |
| Petal.Length | Petal.Width  |
| Petal.Length | Species      |
| Petal.Width  | Species      |

``` r
bulk_cor <- pmap(comb, ~ cor.test(iris[[.x]], iris[[.y]])) %>% 
  map_df(broom::tidy) %>% 
  cbind(comb, .)

knitr::kable(bulk_cor, digits = 3)
```

| V1           | V2           | estimate |     statistic | p.value | parameter | conf.low | conf.high | method                               | alternative |
| :----------- | :----------- | -------: | ------------: | ------: | --------: | -------: | --------: | :----------------------------------- | :---------- |
| Sepal.Length | Sepal.Width  |    1.000 | 816414566.780 |   0.000 |       148 |    1.000 |     1.000 | Pearson’s product-moment correlation | two.sided   |
| Sepal.Length | Petal.Length |    0.872 |        21.646 |   0.000 |       148 |    0.827 |     0.906 | Pearson’s product-moment correlation | two.sided   |
| Sepal.Length | Petal.Width  |  \-0.428 |       \-5.768 |   0.000 |       148 |  \-0.551 |   \-0.288 | Pearson’s product-moment correlation | two.sided   |
| Sepal.Length | Species      |    0.963 |        43.387 |   0.000 |       148 |    0.949 |     0.973 | Pearson’s product-moment correlation | two.sided   |
| Sepal.Width  | Petal.Length |    0.818 |        17.296 |   0.000 |       148 |    0.757 |     0.865 | Pearson’s product-moment correlation | two.sided   |
| Sepal.Width  | Petal.Width  |  \-0.366 |       \-4.786 |   0.000 |       148 |  \-0.497 |   \-0.219 | Pearson’s product-moment correlation | two.sided   |
| Sepal.Width  | Species      |    1.000 | 577292276.430 |   0.000 |       148 |    1.000 |     1.000 | Pearson’s product-moment correlation | two.sided   |
| Petal.Length | Petal.Width  |  \-0.118 |       \-1.440 |   0.152 |       148 |  \-0.273 |     0.044 | Pearson’s product-moment correlation | two.sided   |
| Petal.Length | Species      |    0.818 |        17.296 |   0.000 |       148 |    0.757 |     0.865 | Pearson’s product-moment correlation | two.sided   |
| Petal.Width  | Species      |  \-0.366 |       \-4.786 |   0.000 |       148 |  \-0.497 |   \-0.219 | Pearson’s product-moment correlation | two.sided   |

## Some Machine Learning

### lm

Let’s build a linear model of all possible iris combinations:

``` r
res <- pmap(comb, ~ lm(iris[[.x]] ~ iris[[.y]]))
get_rsquared <- compose(as_mapper(~ .x$r.squared), summary)
map_dbl(res, get_rsquared)
```

    ##  [1] 1.00000000 0.75995465 0.18356092 0.92710984 0.66902769 0.13404820
    ##  [7] 1.00000000 0.01382265 0.66902769 0.13404820

Any chance there’s a r.squared above 0.5 ?

``` r
map(res, get_rsquared) %>% some(~ .x > 0.5)
```

    ## [1] TRUE

### rpart

We’ll build 20 `rpart` to find the better model. Yes, this will be
better to do it with a random forest, but we’re here for the example :)

#### Training and validation

Let’s do the train / validation.

``` r
library(dplyr)
library(readr)
titanic <- read_csv("http://biostat.mc.vanderbilt.edu/wiki/pub/Main/DataSets/titanic3.csv")
train <- rerun(20, sample_frac(titanic, size = 0.8))
validation <- map(train, ~ anti_join(titanic, .x))
```

Check if the sizes of all validation datasets are the same:

``` r
map_int(validation, nrow) %>% every(~ .x == 262)
```

    ## [1] TRUE

### Create a model builder

We’ll create a simple model to predict survival based on sex.

``` r
library(rpart)
rpart_pimped <- partial(rpart, formula = survived ~ sex, method = "class")
res <- map(train, ~ rpart_pimped(data = .x))
```

### Make prediction

``` r
prediction <- map2(validation, res, ~ predict(.y, .x, type = "class"))
w_prediction <- map2(validation, prediction, ~ mutate(.x, prediction = .y))
```

### Confusion matrix

Now let’s map a conf matrix on all these results:

``` r
library(caret)
```

    ## Loading required package: lattice

    ## Loading required package: ggplot2

    ## 
    ## Attaching package: 'caret'

    ## The following object is masked from 'package:purrr':
    ## 
    ##     lift

``` r
conf_mats <- map(w_prediction, ~ confusionMatrix(.x$prediction, .x$survived))
```

Is the “Sensivity” for all models above 0.8?

``` r
map_dbl(conf_mats, ~ .x$byClass["Sensitivity"]) %>% every(~ .x > 0.8)
```

    ## [1] TRUE

Is the “Specificity” for all models above 0.8?

``` r
map_dbl(conf_mats, ~ .x$byClass["Specificity"]) %>% every(~ .x > 0.8)
```

    ## [1] FALSE

### keep\_index

Let’s modify `keep` a little bit we can extract the model we need:

``` r
# Here's the keep source code
keep
```

    ## function(.x, .p, ...) {
    ##   sel <- probe(.x, .p, ...)
    ##   .x[!is.na(sel) & sel]
    ## }
    ## <environment: namespace:purrr>

``` r
# Let's twek it a little bit
keep_index <- function(.x, .p, ...) {
  sel <- purrr:::probe(.x, .p, ...)
  which(sel)
}
```

So, which are the model with a sensitivity superior to
0.85?

``` r
map_dbl(conf_mats, ~ .x$byClass["Sensitivity"]) %>% keep_index(~ .x > 0.85)
```

    ## [1]  1  4  8 20

And the models with a specificity superior to
0.7?

``` r
map_dbl(conf_mats, ~ .x$byClass["Specificity"]) %>% keep_index(~ .x > 0.7)
```

    ## [1]  3  8  9 10 12 13

Which models are in
both?

``` r
sens <- map_dbl(conf_mats, ~ .x$byClass["Sensitivity"]) %>% keep_index(~ .x > 0.85)
spec <- map_dbl(conf_mats, ~ .x$byClass["Specificity"]) %>% keep_index(~ .x > 0.7)
keep(sens, map_lgl(sens, ~ .x %in% spec))
```

    ## [1] 8

So, I guess we’ll go for model number 8\!

![](https://media.giphy.com/media/ohdY5OaQmUmVW/giphy.gif)
