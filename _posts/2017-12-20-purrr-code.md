---
title: "A Crazy Little Thing Called {purrr} - Part 5: code optimization"
author: colin_fay
post_date: 2017-12-20
layout: single
permalink: /purrr-code-optim/
categories : r-blog-en
output: jekyllthat::jekylldown
excerpt_separator: <!--more-->
---

This might not be the last time I make this Queen reference.

<!--more-->

There’s a general saying in programming that you should try to write as
less code as possible. Spoiler: this is not because developers are lazy.
On the contrary, concise code takes a lot of time to write. But it can
save a massive amount of time on the long run, as it creates fewer bugs,
and make the code easier to maintain.

As you know, I love {purrr}. Most tutorials on the web focus on the
iteration part (don’t make me say what I didn’t say: the iteration tools
are incredibly poweful), yet forgetting the “programming” part of
{purrr} would make you miss a lot of cool features from this package. I
hope my series of posts has convince you to dive into this “programming
with {purrr}” world :)

So, today, we’re gonna have another look at what you can do with {purrr}
to write more efficient R.

## Repeat after me: don’t repeat yourself

It has been said countless time: if you copy and paste a piece of code
more than two times, you need a function. Well, I know, it’s not always
that easy to find the perfect function and RStudio makes it so easy to
just copy down the line you’ve just typed. But everytime you wish to do
that, remember that this is just “adding more fuel to the bug fire”.

(Ok, that might not be important if you’re just randomly trying stuffs,
but let’s stay focused and imagine we’re responsable adults building
packages, or doing reproducible research…)

## Functions returning functions

R is an amazingly flexible language. Everything that exists in R is an
object, and everything that happens is due to a function. So yes,
functions are objects. And if a function can take objects and return an
object, there’s no reason these objects can’t be functions. That’s
exactly what `safely` (seen in the first part of this series) and
friends do. But today I’m here to talk about some other functions,
`compose` and
    `partial`.

### Compose

``` r
library(tidyverse)
```

    ## ── Attaching packages ────────────────────────────────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 2.2.1     ✔ purrr   0.2.4
    ## ✔ tibble  1.3.4     ✔ dplyr   0.7.4
    ## ✔ tidyr   0.7.2     ✔ stringr 1.2.0
    ## ✔ readr   1.1.1     ✔ forcats 0.2.0

    ## ── Conflicts ───────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(broom)
```

Have you ever been in a situation when you wrote this kind of
code?

``` r
lm(Sepal.Length ~ Species, data = iris) %>% tidy() %>% filter(p.value < 0.05)
lm(Pepal.Length ~ Species, data = iris) %>% tidy() %>% filter(p.value < 0.05)
lm(Sepal.Width ~ Species, data = iris) %>% tidy() %>% filter(p.value < 0.05)
lm(Sepal.Length ~ Species, data = iris) %>% tidy() %>% ilter(p.value < 0.05)
```

Yes, we’ve all done that. The thing is, with this kind of code, our eyes
are focused on what stays the same, instead of focusing of what’s
changing. And, most of all, there’s a high probability you didn’t
noticed I made a typo on line 2 (example inspired by
<http://r4ds.had.co.nz/iteration.html>).

And there’s also one on line 4.

So here comes `compose()`. This function takes as arguments a series of
functions, and returns a function that will perform the series of
function successively. So here, we can turn :

``` r
tidy(lm(Sepal.Length ~ Species, data = iris))
```

into

``` r
tidy_lm <- compose(tidy, lm)
tidy_lm(Sepal.Length ~ Species, data = iris)
```

    ##                term estimate  std.error statistic       p.value
    ## 1       (Intercept)    5.006 0.07280222 68.761639 1.134286e-113
    ## 2 Speciesversicolor    0.930 0.10295789  9.032819  8.770194e-16
    ## 3  Speciesvirginica    1.582 0.10295789 15.365506  2.214821e-32

So yes, this is a small improvement you could say. But the thing is, on
this kind of code :

``` r
tidy_fancy_calculus <- compose(tidy, lm)
tidy_fancy_calculus(Sepal.Length ~ Species, data = iris)
```

    ##                term estimate  std.error statistic       p.value
    ## 1       (Intercept)    5.006 0.07280222 68.761639 1.134286e-113
    ## 2 Speciesversicolor    0.930 0.10295789  9.032819  8.770194e-16
    ## 3  Speciesvirginica    1.582 0.10295789 15.365506  2.214821e-32

``` r
tidy_fancy_calculus(Petal.Length ~ Species, data = iris)
```

    ##                term estimate  std.error statistic      p.value
    ## 1       (Intercept)    1.462 0.06085848  24.02294 9.303052e-53
    ## 2 Speciesversicolor    2.798 0.08606689  32.50960 5.254587e-69
    ## 3  Speciesvirginica    4.090 0.08606689  47.52118 4.106139e-91

``` r
tidy_fancy_calculus(Sepal.Width ~ Species, data = iris)
```

    ##                term estimate  std.error statistic       p.value
    ## 1       (Intercept)    3.428 0.04803910 71.358540 5.707614e-116
    ## 2 Speciesversicolor   -0.658 0.06793755 -9.685366  1.832489e-17
    ## 3  Speciesvirginica   -0.454 0.06793755 -6.682608  4.538957e-10

``` r
tidy_fancy_calculus(Petal.Width ~ Species, data = iris)
```

    ##                term estimate  std.error statistic      p.value
    ## 1       (Intercept)    0.246 0.02894188  8.499792 1.959455e-14
    ## 2 Speciesversicolor    1.080 0.04093000 26.386510 1.254978e-57
    ## 3  Speciesvirginica    1.780 0.04093000 43.488878 7.951748e-86

I’ll just have to change at one place if I need to use another `lm`-like
function.

Note that the order of the function is not the a magrittr order, but a
“classic” order, i.e. functions passed to `compose` are executed from
right to left.

And of course, you can always pass mappers in a `compose` :

``` r
clean_lm <- compose(as_mapper(~ arrange(.x, desc(p.value))), 
                    as_mapper(~ filter(.x, p.value < 0.05)),
                    tidy, 
                    lm)
clean_lm(Sepal.Length ~ Sepal.Width, data = iris)
```

    ##          term estimate std.error statistic      p.value
    ## 1 (Intercept) 6.526223 0.4788963  13.62763 6.469702e-28

``` r
# and etc
#clean_lm(Sepal.Length ~ Sepal.Width, data = iris)
#clean_lm(Sepal.Length ~ Sepal.Width, data = iris)
#clean_lm(Sepal.Length ~ Sepal.Width, data = iris)
```

For now formulas are not accepted as such (you need to call
`as_mapper`), but you now, this is just a PR away ;)

### Less code, more rokk

[(Guitar Hero 2 reference)](https://www.youtube.com/watch?v=q2eBtnxA8SA)

So now, we need even less code. Yes, that’s possible, thanks to
`partial`. As its name states, this function returns a partially filled
function. That is to say that :

``` r
mean_na_rm <- partial(mean, na.rm = TRUE)
mean_na_rm(airquality$Ozone)
```

    ## [1] 42.12931

is the same as:

``` r
mean(airquality$Ozone, na.rm = TRUE)
```

    ## [1] 42.12931

Ok, that’s not that usefull on that example, but you get the idea :)

### Let’s wrap up

To sum up, we need a function that will do a lm on iris, broom::tidy the
result, filter the p.value, and arrange following the p.value column. So
:

``` r
clean_lm_iris <- compose(as_mapper(~ arrange(.x, desc(p.value))), 
                    as_mapper(~ filter(.x, p.value < 0.05)),
                    tidy, 
                    partial(lm, data = iris))

clean_lm_iris(Sepal.Length ~ Sepal.Width)
```

    ##          term estimate std.error statistic      p.value
    ## 1 (Intercept) 6.526223 0.4788963  13.62763 6.469702e-28
