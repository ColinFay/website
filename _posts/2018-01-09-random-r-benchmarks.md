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

There are two main things you can do to optimise your code: **write more
and shorter functions** to avoid repeating yourself
(maintainance-oriented optimisation), and **find ways to make your R
code run faster** (usage-oriented optimisation).

One way to make you R code run really faster is to turn to languages
like C++. But you know, [it’s not that
easy](https://memegenerator.net/img/instances/500x/39604158/i-love-c-it-makes-people-cry.jpg),
and before turning to that, you can still use some tricks in plain R.

I was working on {attempt} lately, a package that (I hope) makes
conditions handling easier, and aims to be used inside other functions
([more on that](https://github.com/ColinFay/attempt)). As the {attempt}
functions are to be used inside other functions, I focused on trying to
speed things up as much as I could, in order to keep the general UX
attractive. And of course, I wanted to do that without having to go for
C / C++ (well, mainly because I don’t know how to program in C).

For this, I played a lot with {microbenchmark}, a package that allows to
**compare the time spent to run several R commands**, and in the end, I
was glad to see that using these benchmarks allowed me to write faster
functions (and sometimes, really faster). I won’t share them all here
(because this is a one thousand lines script), but here are some pieces
taken out of it.

> Because at the end of the day, someone always asks: “and what about
> performance?”

## return or stop as soon as possible

Funny thing: as I started writing this post, I came accross [this
blogpost](https://yihui.name/en/2018/01/stop-early/) by Yihui that
states the exact same thing I was writing: **focus on `return()` or
`stop()` as soon as possible**. Don’t make your code compute stuffs you
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
    ##  compute_first(1) 474 500.5 574.22  525.0 568.5 4042   100   b
    ##       if_first(1) 234 248.0 307.92  262.5 275.5 3938   100  a 
    ##  compute_first(0) 465 492.0 530.41  504.5 551.0  790   100   b
    ##       if_first(0) 468 503.5 541.88  513.5 559.5 1880   100   b

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
    ##             expr      min         lq        mean     median         uq
    ##   sleep_first(1) 10027645 10554083.5 11506608.74 11451679.5 12573032.5
    ##  sleep_second(1)      433      981.5     6369.33     6727.5     8333.5
    ##   sleep_first(0) 10028171 10522980.0 11616219.55 11593884.5 12681759.0
    ##  sleep_second(0) 10039724 10485246.5 11655079.38 11966281.5 12675001.0
    ##       max neval cld
    ##  12722973   100   b
    ##     77447   100  a 
    ##  12706475   100   b
    ##  12751273   100   b

## Use `importFrom`, not `::`

Yes, `::` is a function, so calling `pkg::function` is slower than
attaching the package and then use the function. That’s why when
creating a package, it’s better to use `@importFrom`. Do this instead of
just listing the package as dependency, and then using `::` inside your
functions.

We see this even with base:

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
    ##              expr  min   lq      mean median     uq     max neval cld
    ##        is.null(a)   79  101  149.6044    113  155.0   27636 10000  a 
    ##  base::is.null(a) 3522 3822 5147.8318   4129 5799.5  161543 10000   b
    ##          is.na(b)  105  130  189.1814    143  190.0   26889 10000  a 
    ##    base::is.na(b) 3575 3891 5433.2070   4197 5929.0 1918760 10000   b

As a general rule of thumb, you should aim at **making as less function
calls as possible**: calling one function takes time, so if can be
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
    ##     with_return(3) 223 236 375.2277    242 256 1130331 10000   a
    ##  without_return(3) 224 236 335.7968    242 258  743385 10000   a

Surprisingly, `return()` doesn’t slow your code. So use it\!

## Brackets make the code (a little bit) slower

But they make it way clearer. So **keep it unless you want to win a
bunch of nanoseconds** :)

``` r
microbenchmark(if(TRUE)"yay",
               if(TRUE) {"yay"},
               if(FALSE) "yay" else "nay", 
               if(FALSE) {"yay"} else {"nay"}, 
               times = 10000)
```

    ## Unit: nanoseconds
    ##                                         expr min lq    mean median uq  max
    ##                              if (TRUE) "yay"  43 45 48.9238     48 50  305
    ##                      if (TRUE) {     "yay" }  79 82 88.5321     89 91  522
    ##                  if (FALSE) "yay" else "nay"  44 50 54.6678     53 59 1918
    ##  if (FALSE) {     "yay" } else {     "nay" }  81 89 91.7045     90 95 1736
    ##  neval  cld
    ##  10000 a   
    ##  10000   c 
    ##  10000  b  
    ##  10000    d

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
    ##     with_assign(3) 444 472 3409.123    493 550 2787270  1000   a
    ##  without_assign(3) 278 300 2871.232    311 358 2428304  1000   a

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
    ##       with_pipe(3) 93317 96293.5 123352.207 99720.5 113969.5 2759033  1000
    ##  without_assign(3)   279   324.0    658.431   512.5    601.0   52923  1000
    ##  cld
    ##    b
    ##   a

You know, because `%>%` is a function.

## Conclusion

**These tests are just random tests I run**, and I can’t say they cover
all the technics you can use to speed up your code: rhese are just some
questions I can accross and wanted to share.

At the end of the day, there are also methods (like the bracket one)
that can help you save some nanoseconds. So don’t be afraid of not using
them, and stay focus on what’s more important: **write code that is easy
to understand. Because a piece of code that is easy to understand is a
piece of code that is easy to maintain\!**

If you want to read more about code optimisation for speed, Colin
Gillespie and Robin Lovelace wrote a nice book about being more efficien
with R, with a chapter focused on performance: [Efficient
optimisation](https://csgillespie.github.io/efficientR/performance.html)
