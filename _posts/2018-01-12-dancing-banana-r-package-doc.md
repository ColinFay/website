---
title: "[How to] Include a dancing banana in your R package documentation"
author: colin_fay
post_date: 2018-01-12
layout: single
permalink: /dancing-banana-r-package-doc/
categories: r-blog-en
output: jekyllthat::jekylldown
excerpt_separator: <!--more-->
---

Having fun playing around with R documentation and HTML.

<!--more-->

I found out today that you could [easily insert html and
JS](https://twitter.com/_ColinFay/status/951838799918755842) into R help
pages, as I walked into Clippy when reading the [{writexl} package
documentation](https://twitter.com/_ColinFay/status/951819899248283651).

So, if like me you’re curious about how, here’s a short how-to.

## Playing with the .Rd

A package documentation is composed of .Rd files, which lives (when
developping) in the `/man` folder of your source package. They are made
of LaTeX, but nowadays we rarely write LaTeX by hand, but rely on
{roxygen2} for doing the heavy lifting for package documentation. But
still, you can write LaTeX code into the roxygen comments. That’s the
trick I’m gonna used, borrowed to Jeroen {writexl}’s code.

I saw it was possible to insert arbitrary HTML and JS by using, as a
roxygen comment, a combination of:

``` r
#' \if{html}{
#' \out{
#' Your HTML here
#' }}
```

With that, the HTML/JS code just pops up at the bottom of the
documentation, when doing `?myfun`.

So basically, if I put :

``` r
#' \if{html}{
#' \out{
#'  <img src = "https://media.giphy.com/media/IB9foBA4PVkKA/giphy.gif">
#' }}
#'
```

The doc goes:

![](img/dancing-banana-r-package.gif)

I don’t know LaTeX, so I can’t perfectly explain how it works, but the
code is pretty straightforward, so we can easily guess the `\if{html}`
tests if the code is run in an html page, and if it is, the HTML code is
inserted. If this is not what happens, I’ll be glad to here some
explanation about that, so feel free to comment below.

## Insert YouTube video

Of course, this could be more useful, like, putting a gif with a code
example, or a video with a tutorial.

But I guess it’s still better to get
[rickrolled](https://www.youtube.com/watch?v=dQw4w9WgXcQ), so here’s the
code to do that (and if you have mercy for your users, turn the autoplay
off by changing to `autoplay=0` ;) ):

``` r
#' \if{html}{
#' \out{
#'<iframe width="420" height="345" src="http://www.youtube.com/embed/dQw4w9WgXcQ?autoplay=1" frameborder="0" allowfullscreen></iframe>
#' }}
#'
```

![](img/rick-rolled.gif)
