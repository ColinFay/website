---
title: "Some random R benchmarks"
author: colin_fay
post_date: 2018-01-09
layout: single
permalink: /random-r-benchmarks/
categories: cat
output: jekyllthat::jekylldown
excerpt_separator: <!--more-->
---

Some random (one could say useless) benchmarks on R, serious or for fun,
and some advices to make your code run faster. <!--more-->

There are two main things you can do to optimise code: write shorter
functions to avoid repeating yourself (maintainance-oriented
optimisation), and find how to make your R code run faster
(usage-oriented optimisation).

One way to make you R code run really faster is to turn to languages
like C++. But you know, [it’s not that
easy](https://memegenerator.net/img/instances/500x/39604158/i-love-c-it-makes-people-cry.jpg),
and before turning to that, you can still use some tricks to help your
code run faster.

I was working on {attempt} lately, a package that (I hope) makes
conditions handling easier, and aims to be used inside other functions
([more on that](https://github.com/ColinFay/attempt)). As the {attempt}
functions are to be used inside other functions, I focused on trying to
speed things up as much as I could, in order to keep the general UX
attractive. And of course, I wanted to do that without having to go for
C / C++ (well, mainly because I don’t know how to program in C).

For this, I played a lot with {microbenchmark}, a package that allows to
compare the time spent to run several R commands, and in the end, I was
glad to see that using these benchmarks allowed me to write faster
functions (and sometimes, really faster). I won’t share them all here
(because this is one thousand lines of code), but here are some pieces
taken out of it.

Because at the end of the day, someone always asks: “and what about
performance?”

## return or stop as soon as possible

Funny thing: as I started writing this post, I came accross [this
blogpost](https://yihui.name/en/2018/01/stop-early/) by Yihui that
states the exact same thing I was writing: focus on `return()` or
`stop()` as soon as possible. Don’t make your code compute stuffs you
might never need in some cases, or make useless tests. For example, if
you have an if statement that will stop the function, test it first.

> Note: the examples that follow are of course toy examples.

``` r
library(microbenchmark)

compute_first <- function(a){
  b <- log(a) * 10 
  c <- log(a) * 100
  d <- b + c
  if( a == 1 ) {
    return(0)
  }
  return(d)
}
if_first <- function(a){
  if( a == 1 ) {
    return(0)
  }
  b <- log(a) * 10 
  c <- log(a) * 100
  d <- b + c
  return(d)
}

# First, make sure you're benchmarking on the same result
all.equal(compute_first(3), if_first(3))
```

    ## [1] TRUE

``` r
all.equal(compute_first(0), if_first(0))
```

    ## [1] TRUE

``` r
microbenchmark(compute_first(1), if_first(1), compute_first(0), if_first(0), times = 100)
```

    ## Unit: nanoseconds
    ##              expr min    lq   mean median    uq  max neval cld
    ##  compute_first(1) 473 510.5 588.45  544.0 608.0 1580   100   b
    ##       if_first(1) 236 250.0 318.03  266.5 315.0 2993   100  a 
    ##  compute_first(0) 471 505.5 659.36  536.0 623.0 8869   100   b
    ##       if_first(0) 471 505.0 576.13  529.5 600.5 1309   100   b

As you can see, `compute_first` takes almost as much time to compute in
both cases. But you can save some time when using `if_first` : in the
case of `a == 1`, the function doesn’t compute `b`, `c` or `d`, so runs
faster.

Yes, it’s a matter of nanoseconds, but what if:

``` r
sleep_first <- function(b){
  Sys.sleep(0.01)
  if (b == 1 ){
    return(0)
  }
  return(b * 10)
}
sleep_second <- function(b){
  if (b == 1 ){
    return(0)
  }
  Sys.sleep(0.01)
  return(b * 10)
}

all.equal(sleep_first(1), sleep_second(1))
```

    ## [1] TRUE

``` r
all.equal(sleep_first(0), sleep_second(0))
```

    ## [1] TRUE

``` r
microbenchmark(sleep_first(1), sleep_second(1), sleep_first(0), sleep_second(0), times = 100)
```

    ## Unit: nanoseconds
    ##             expr      min       lq        mean     median       uq
    ##   sleep_first(1) 10042022 10615753 11545262.42 11578319.0 12673020
    ##  sleep_second(1)      594     5607     7494.75     7376.5     8338
    ##   sleep_first(0) 10041663 10443558 11531655.29 11485839.5 12679463
    ##  sleep_second(0) 10072008 10660272 11477697.63 11375085.5 12673500
    ##       max neval cld
    ##  12775573   100   b
    ##     59000   100  a 
    ##  12848165   100   b
    ##  12736441   100   b

## Use `importFrom`, not `::`

Yes, `::` is a function, so calling `pkg::function` is slower than
attaching the package and then use the function. That’s why \<hen
creating a package, it’s better to use `@importFrom`. Do this instead of
just listing the package as dependency, and then using `::` inside your
functions.

And that’s not specific to some cases, we get the same results with
base:

``` r
a <- NULL
b <- NA
microbenchmark(is.null(a), 
               base::is.null(a), 
               is.na(b), 
               base::is.na(b),
               times = 10000)
```

    ## Unit: nanoseconds
    ##              expr  min   lq      mean median   uq     max neval cld
    ##        is.null(a)   76  103  141.2799    115  132   25371 10000  a 
    ##  base::is.null(a) 3671 4065 5160.2074   4337 4786  345280 10000   b
    ##          is.na(b)  106  137  181.1913    148  176   37625 10000  a 
    ##    base::is.na(b) 3721 4105 5475.4224   4381 4822 2596615 10000   b

As a general rule of thumb, you should aim at making as less function
calls as possible: calling one function takes time, so if can be
avoided, avoid it.

Here, for example, there’s a way to avoid calling `::`, so use it :)

## Does `return()` make the funcion slower?

I’ve heard several times that `return()` slows your code a little bit
(that makes sens, return is a function call).

``` r
with_return <- function(x){
  return(x*1e3)
}

without_return <- function(x){
  x*1e3
}

all.equal(with_return(3), without_return(3))
```

    ## [1] TRUE

``` r
microbenchmark(with_return(3),
               without_return(3),
               times = 10000)
```

    ## Unit: nanoseconds
    ##               expr min  lq     mean median  uq     max neval cld
    ##     with_return(3) 224 241 453.9917    263 394 1110423 10000   a
    ##  without_return(3) 224 239 474.9781    261 392 1369807 10000   a

Surprisingly, `return()` doesn’t slow your code. So use it\!

## Brackets make the code (a little bit) slower

But they make it way clearer. So keep it unless you want to win a bunch
of nanoseconds :)

``` r
microbenchmark(if(TRUE)"yay",
               if(TRUE) {"yay"},
               if(FALSE) "yay" else "nay", 
               if(FALSE) {"yay"} else {"nay"}, 
               times = 10000)
```

    ## Unit: nanoseconds
    ##                                         expr min  lq     mean median  uq
    ##                              if (TRUE) "yay"  38  55  88.7599     81  99
    ##                      if (TRUE) {     "yay" }  75  95 171.0706    152 181
    ##                  if (FALSE) "yay" else "nay"  39  61 105.2843     90 105
    ##  if (FALSE) {     "yay" } else {     "nay" }  77 100 187.1689    160 188
    ##     max neval cld
    ##   30012 10000  a 
    ##   52953 10000   b
    ##  100904 10000  a 
    ##  223661 10000   b

## Assign as less as possible

Yep, because assigning is a function.

``` r
with_assign <- function(a){
  b <- a*10e3
  c <- sqrt(b)
  d <- log(c)
  d
}
without_assign <- function(a){
  log(sqrt(a*10e3))
}

microbenchmark(with_assign(3),
               without_assign(3),
               times = 1000)
```

    ## Unit: nanoseconds
    ##               expr min  lq     mean median  uq     max neval cld
    ##     with_assign(3) 440 463 2163.756    479 510 1666819  1000   a
    ##  without_assign(3) 274 289 2273.333    299 307 1897253  1000   a

## What about the pipe ?

Does the pipe makes things slower?

``` r
library(magrittr)
with_pipe <- function(a){
  10e3 %>% "*"(a) %>% sqrt() %>% log()
}

all.equal(with_pipe(3), without_assign(3))
```

    ## [1] TRUE

``` r
microbenchmark(with_pipe(3),
               without_assign(3),
               times = 1000)
```

    ## Unit: nanoseconds
    ##               expr   min      lq       mean  median       uq     max neval
    ##       with_pipe(3) 94538 97644.5 115050.857 99866.0 111293.5 2589781  1000
    ##  without_assign(3)   279   316.5    508.961   488.5    564.5    3220  1000
    ##  cld
    ##    b
    ##   a

You know, because `%>%` is a function.

## Conclusion

These tests are just random tests I run, and I can’t say they cover all
the technics you can use to speed up your code a little bit. These are
just some questions I can accross and wanted to share.

At the end of the day, there are also methods (like the bracket one)
that can help you save some nanoseconds. So don’t be afraid of not using
them, and stay focus on what’s more important: write code that is easy
to understand. Because a piece of code that is easy to understand is a
piece of code that is easy to maintain\!

If you want to read more about code optimisation for speed, Colin
Gillespie and Robin Lovelace wrote a nice book about being more efficien
with R, with a chapter focused on performance: [Efficient
optimisation](https://csgillespie.github.io/efficientR/performance.html)
