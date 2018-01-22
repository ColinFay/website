---
title: "Can (a==1 && a==2 && a==3) ever evaluate to true?"
author: colin_fay
post_date: 2018-01-22
layout: single
permalink: /active-binding-r-enigma/
categories: r-blog-en
output: jekyllthat::jekylldown
excerpt_separator: <!--more-->
---

A blogpost inspired by a Stackoverflow question: “Can (a==1 && a==2 &&
a==3) ever evaluate to true?”

<!--more-->

Yesterday I run into this [topic on
SO](https://stackoverflow.com/questions/48270127/can-a-1-a-2-a-3-ever-evaluate-to-true),
about a JavaScript question a user had during a job interview.

The question is quite straightforward:

> Is it ever possible that (a==1 && a==2 && a==3) could evaluate to
> true, in JavaScript?

As a little challenge, I asked myself the same question, but with R: can
it ever evaluate to TRUE? Once I’ve found how to, I [challenged
Twitter](https://twitter.com/_ColinFay/status/955116156666499072), and
was glad to read some creative answers.

One amazing thing with R is that you can achieve the same result by
taking several roads. So, here are a compilation of the answers I
received.

## First question

<blockquote class="twitter-tweet" data-lang="en">

<p lang="en" dir="ltr">

Ok
<a href="https://twitter.com/hashtag/RStats?src=hash&amp;ref_src=twsrc%5Etfw">\#RStats</a>
nerds, here's a little enigma for your sunday :) <br>How can you
make:<br>x == 1 & x == 2 & x == 3<br>return<br>\[1\] TRUE
<a href="https://t.co/toV55YhjQJ">pic.twitter.com/toV55YhjQJ</a>

</p>

— Colin Fay (@\_ColinFay)
<a href="https://twitter.com/_ColinFay/status/955116156666499072?ref_src=twsrc%5Etfw">January
21,
2018</a>

</blockquote>

<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

### Overriding base functions

  - By [Philippe Chataignon](https://twitter.com/phchataignon) :

<!-- end list -->

``` r
x <- 1
`&` <- function(x,y) {T}
x == 1 & x == 2 & x == 3
```

    ## [1] TRUE

  - By [Quentin Read](https://twitter.com/QuentinDRead) :

<!-- end list -->

``` r
`&` <- function(x,y) {T}
x == 1 & x == 2 & x == 3
```

    ## [1] TRUE

## Number 2

<blockquote class="twitter-tweet" data-lang="en">

<p lang="en" dir="ltr">

<a href="https://twitter.com/hashtag/RStats?src=hash&amp;ref_src=twsrc%5Etfw">\#RStats</a>
nerd quizz number 2 : <br>How would you code that?<br>\> x == 1 & x == 2
& x == 3<br>\[1\] TRUE<br>\> x == 3 & x == 2 & x == 1<br>\[1\] FALSE
<a href="https://t.co/Tqm0bNz1De">pic.twitter.com/Tqm0bNz1De</a>

</p>

— Colin Fay (@\_ColinFay)
<a href="https://twitter.com/_ColinFay/status/955137171605860352?ref_src=twsrc%5Etfw">January
21,
2018</a>

</blockquote>

<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

### Overriding base functions

  - By [Antonios K](https://twitter.com/tonkouts) :

<!-- end list -->

``` r
x = 1; `&` <- `>`
x == 1 & x == 2 & x == 3
```

    ## [1] TRUE

``` r
x == 3 & x == 2 & x == 1
```

    ## [1] FALSE

  - By [Jorge Loria](https://twitter.com/jelorias95) :

<!-- end list -->

``` r
x <-1
`&` <- function(e1,e2) e1
x == 1 & x == 2 & x == 3
```

    ## [1] TRUE

``` r
x == 3 & x == 2 & x == 1
```

    ## [1] FALSE

## Question 3

<blockquote class="twitter-tweet" data-lang="en">

<p lang="en" dir="ltr">

<a href="https://twitter.com/hashtag/RStats?src=hash&amp;ref_src=twsrc%5Etfw">\#RStats</a>
nerd quizz number 3 : <br>Find how to: <br>\> x<br>\[1\] 1<br>\>
x<br>\[1\] 2<br>\> x<br>\[1\] 3
<a href="https://t.co/dskLLHdS6d">pic.twitter.com/dskLLHdS6d</a>

</p>

— Colin Fay (@\_ColinFay)
<a href="https://twitter.com/_ColinFay/status/955156032728256515?ref_src=twsrc%5Etfw">January
21,
2018</a>

</blockquote>

<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

  - By [Thomas Mailund](https://twitter.com/ThomasMailund)

> A really nice way to frame it

``` r
make_x <- function(){
  counter <- 1
  inc <- function() counter <<- counter + 1
  value <- function() counter
  structure(list(inc = inc, value = value), class = "x")
}
print.x <- function(x){
  v <-x$value()
  x$inc()
  print(v)
  v
}
x <- make_x()
x
```

    ## [1] 1

``` r
x 
```

    ## [1] 2

``` r
x
```

    ## [1] 3

  - As suggested by [Joel Gombin](https://twitter.com/joelgombin)

<!-- end list -->

``` r
rm(list=ls())
library(R6)
make_x <- R6Class("x", 
                  public = list(x = 1,
                                print = function(){
                                  print(self$x)
                                  self$x <- self$x + 1
                                }
                  )
)
x <- make_x$new()
x
```

    ## [1] 1

``` r
x
```

    ## [1] 2

``` r
x
```

    ## [1] 3

Sadly, these two solutions don’t answer the three questions in one shot.

## One answer to rule them all

One answer that can do the three at once, by [Alek Vladimir
Nevski](https://twitter.com/AlekVladNevski).

``` r
x <- c(1,1)
`==` <- function(foo,bar){foo[1]+bar}
`&` <- function(foo,bar){
  if(isTRUE(foo) || identical(foo,FALSE)){
  foo+2<bar
  }else{
      foo < bar
    }
  }

class(x) <- c("myclass",class(x)) 
print.myclass = function(x){
  t <- x[2];
  x[2] <<- x[2]+1;print(t)
  }
# First quizz :
x == 1 & x == 2 & x == 3
```

    ## [1] TRUE

``` r
# Second quizz : 
x == 1 & x == 2 & x == 3
```

    ## [1] TRUE

``` r
x == 3 & x == 2 & x == 1
```

    ## [1] FALSE

``` r
# Third quizz :
x
```

    ## [1] 1

``` r
x
```

    ## [1] 2

``` r
x
```

    ## [1] 3

## With active binding

What I had in mind from the beginning was to use active binding. I’ll
let you dig into [this blogpost](http://colinfay.me/ractivebinfing/) if
ever you want to know more about it, but basically we bind a function to
a symbol instead of a value (which is static binding).

It is called active because each time you evaluate this symbol, R
executes a function, instead of looking for a value.

So, here’s my way to answer the “Can (a==1 && a==2 && a==3) ever
evaluate to true?”:

``` r
rm(list=ls())
makeActiveBinding(sym = "x", env = .GlobalEnv,
                  fun = function(value){
                    if (missing(value)) {
                      if (i == 3){
                        i <<- 1
                      } else {
                        i <<- i + 1
                      }
                      return(i)
                    } 
                  }
)
i <- 0
# First quizz :
x == 1 & x == 2 & x == 3
```

    ## [1] TRUE

``` r
# Second quizz : 
x == 1 & x == 2 & x == 3
```

    ## [1] TRUE

``` r
x == 3 & x == 2 & x == 1
```

    ## [1] FALSE

``` r
# Third quizz :
x
```

    ## [1] 1

``` r
x
```

    ## [1] 2

``` r
x
```

    ## [1] 3

Here, the difference is that the values are actually equal: we’re not
changing the way `==` or `&` behave - it’s the value of `x` that is
changing each time we’re calling it. With active binding, R is executing
the function defined in `fun`, a function that takes i, restarts the
counter or incremente it of 1, and returns it.

## How is this interesting?

Beside being a nice R brain game, this example is a nice occasion to
introduce about the concept of binding, and remind that we are most of
the time assuming is static: most of the answers (in fact, all except
one) assume that the value of `x` is static, and that we should tweak
the other operators :)

Read more about binding:

  - [R Programming for Data
    Science](https://bookdown.org/rdpeng/rprogdatascience/scoping-rules-of-r.html)

  - [Robert Gentleman and Ross Ihaka (2000). “Lexical Scope and
    Statistical
    Computing”](https://www.stat.auckland.ac.nz/~ihaka/downloads/lexical.pdf)

  - [Advanced R -
    Environments](http://adv-r.had.co.nz/Environments.html#binding)

  - [Binding and Environment Locking, Active
    Bindings](https://stat.ethz.ch/R-manual/R-devel/library/base/html/bindenv.html)
