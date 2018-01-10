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

Because someone always asks: “and what about performance?”

<!--more-->

There are two main things you can do to optimise code: writting shorter
functions to avoid repeating yourself (maintainance-oriented
optimisation), and finding how to make your R code run faster
(usage-oriented optimisation).

One way to make you R code run really faster is to turn to language like
C++. But you know, [it’s not that
easy](https://memegenerator.net/img/instances/500x/39604158/i-love-c-it-makes-people-cry.jpg),
and before turning to that, you can still use some tricks to help your
code run faster.

I was working on {attempt} lately, a package that helps conditions
handling, aims to be used inside other functions ([more on
that](https://github.com/ColinFay/attempt)). As the attempt functions
are to be used inside other function, I focused on trying to speed
things up as much as I could, in order to keep the general UX
attractive. And of course, I wanted to do that without having to go for
C / C++ (well, mainly because I don’t know how to program in C). For
this, I played a lot with {microbenchmark}, a package that allows to
compare the time spent to run several R commands. In the end, these
benchmarks allowed me to write faster functions. I won’t share them all
here (because this is mainly one thousand lines of specific code), but I
guess I can share some pieces of it.

So, here are some random (one could say useless) benchmarks about R,
serious or for fun, and some advices to make your code run faster.

## return or stop as soon as possible

Funny thing: as I started writing this post, I came accross [this
blogpost](https://yihui.name/en/2018/01/stop-early/) by Yihui that
states the exact same thing I was writting: focus on `return()` or
`stop()` as soon as possible. Don’t make your code compute stuff you
might never need in some cases, or compute realise useless tests. For
example, if you have an if statement that will stop the function, test
it first.

> Note: the examples here are of course toy example.

For example :

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
    ##              expr min    lq   mean median    uq   max neval cld
    ##  compute_first(1) 474 510.0 594.87  560.5 635.5  1503   100   b
    ##       if_first(1) 233 254.5 319.29  278.0 344.0  1601   100  a 
    ##  compute_first(0) 471 513.0 689.10  552.5 608.5 10739   100   b
    ##       if_first(0) 471 510.0 566.42  541.0 600.0  1121   100   b

As you can see, `compute_first` takes almost as much time to compute in
both case. But you can save some time when using `if_first` : in the
case a == 1, the function doesn’t compute `b`, `c` or `d`, so runs
faster. Yes, it’s a matter of nanoseconds, but what if:

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
    ##             expr      min       lq        mean   median         uq
    ##   sleep_first(1) 10037123 10454785 11622239.94 11991320 12674972.0
    ##  sleep_second(1)      258     5288     8935.88     7872     9349.5
    ##   sleep_first(0) 10044047 10590848 11542020.08 11574358 12675459.0
    ##  sleep_second(0) 10039071 10382517 11461044.34 11409743 12636210.0
    ##       max neval cld
    ##  12690190   100   b
    ##     96180   100  a 
    ##  12704257   100   b
    ##  12726656   100   b

## Use importFrom, not `::`

Yes, `::` is a function, so calling `pkg::function` is slower than
attaching the package and then use the function. When creating a
package, it’s better to use `@importFrom`, instead of just listing the
package as dependency, and then using `::` inside your function.

And that’s not specific, we get the same results with base:

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
    ##              expr  min   lq     mean median   uq     max neval cld
    ##        is.null(a)   79   99  144.057    110  135   36615 10000  a 
    ##  base::is.null(a) 3532 3834 5013.332   4159 4759  278601 10000   b
    ##          is.na(b)  105  129  174.649    140  165   11467 10000  a 
    ##    base::is.na(b) 3580 3907 5218.029   4232 4819 1297422 10000   b

As a general rule of thumb, you should aim at doing as less function
calls as possible: calling one function takes time, so if can be
avoided, avoid it. Here, for example, there’s a way to avoid calling
`::`, so use it :)

## Does return() make the funcion slower?

There’s a saying in the R ommunity that `return()` slows a little bit
your code (that makes sens, return is a function call).

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
    ##               expr min  lq     mean median  uq    max neval cld
    ##     with_return(3) 223 236 345.0267    241 257 825418 10000   a
    ##  without_return(3) 221 236 353.0534    240 247 944481 10000   a

Surprisingly, `return()` doesn’t slow your code. So use it\!

## Brackets make the code (a little bit) slower

But way clearer. So keep it unless you want to win a bunch of
nanoseconds :)

``` r
microbenchmark(if(TRUE)"yay",
               if(TRUE) {"yay"},
               if(FALSE) "yay" else "nay", 
               if(FALSE) {"yay"} else {"nay"}, 
               times = 10000)
```

    ## Unit: nanoseconds
    ##                                         expr min lq    mean median  uq
    ##                              if (TRUE) "yay"  38 50 51.3966     52  55
    ##                      if (TRUE) {     "yay" }  75 87 91.8743     92  95
    ##                  if (FALSE) "yay" else "nay"  39 54 56.3569     56  61
    ##  if (FALSE) {     "yay" } else {     "nay" }  77 91 98.4189     96 101
    ##   max neval  cld
    ##   215 10000 a   
    ##  8379 10000   c 
    ##  1078 10000  b  
    ##  8018 10000    d

## Assign as less as possible

Yep, because assigning is a function (still, a matter of nanoseconds).

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
    ##               expr  min   lq     mean median     uq     max neval cld
    ##     with_assign(3) 1097 1143 5171.114   1177 1218.0 3945914  1000   a
    ##  without_assign(3)  714  751 3725.098    777  803.5 2916320  1000   a

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
    ##               expr   min    lq       mean   median       uq     max neval
    ##       with_pipe(3) 94652 97425 131748.555 102394.5 147620.0 2265369  1000
    ##  without_assign(3)   280   344    615.917    520.0    640.5   10454  1000
    ##  cld
    ##    b
    ##   a

You know, because `%>%` is a function.

## Conclusion

These tests are just random tests I run, and I can’t say they cover all
the technics you can use to speed up your code a little bit.

If you want to read more about code optimisation for speed, Colin
Gillespie and Robin Lovelace wrote a nice book about being more efficien
with R, with a chapter focused on performance: [Efficient
optimisation](https://csgillespie.github.io/efficientR/performance.html)
