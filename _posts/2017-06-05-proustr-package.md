---
title: "New R Package — proustr"
author: colin_fay
post_date: 2017-06-05
layout: single
permalink: /proustr-package/
categories : r-blog-en
excerpt_separator: <!--more-->
---

`proustr` is now on [CRAN](https://cran.r-project.org/web/packages/proustr/index.html).

## An R Package for Marcel Proust's A La Recherche Du Temps Perdu

This package gives you access to all the books from Marcel Proust "À la recherche du temps perdu" collection. This collection is divided in books, which are divided in volumes. Inspired by the package [janeaustenr](https://github.com/juliasilge/janeaustenr) by Julia Silge. 

All books have been downloaded from [BEQ](https://beq.ebooksgratuits.com/auteurs/Proust/proust.htm) 

Here is a list of all the books contained in this pacakage : 

+ Du côté de chez Swann (1913): 2 volumes `ducotedechezswann1` & `ducotedechezswann2`. 
+ À l'ombre des jeunes filles en fleurs (1919): 3 volumes `alombredesjeunesfillesenfleurs1`, `alombredesjeunesfillesenfleurs2`, and `alombredesjeunesfillesenfleurs3`.
+ Le Côté de Guermantes (1920-1921): 3 volumes `lecotedeguermantes1`, `lecotedeguermantes2` and `lecotedeguermantes3`
+ Sodome et Gomorrhe I et II (1921-1922) : 2 volumes `sodomeetgomorrhe1` and `sodomeetgomorrhe2`.
+ La Prisonnière (1923) : 2 volumes `laprisonniere1` and `laprisonniere2`.
+ Albertine disparue (1925, also know as : La Fugitive) : `albertinedisparue`.
Le Temps retrouvé (1927) : 2 volumes `letempretrouve1` and `letempretrouve2`.


## Install proustr

Install this package directly in R : 

From CRAN :

{% highlight r %}
install.packages("proustr")
{% endhighlight %}

From Github :

{% highlight r %}
devtools::install_github("ColinFay/proustr")
{% endhighlight %}

## Examples 

{% highlight r %}
devtools::install_github("ThinkRstat/stopwords")
library(proustr)
library(tidytext)
library(tidyverse)
library(stopwords)
books <- proust_books()
books %>%
  unnest_tokens(word, text) %>%
  filter(word %in% stopwords_iso$fr) %>%
  count(word, sort = TRUE)%>%
  head(10)
{% endhighlight %}

{% highlight r %}
# A tibble: 10 x 2
      word     n
     <chr> <int>
 1   qu’il  5409
 2 qu’elle  4088
 3   c’est  3533
 4    d’un  3440
 5     mme  3106
 6   d’une  2920
 7   faire  2869
 8   qu’on  2560
 9 j’avais  2089
10 c’était  1858
{% endhighlight %}

### Contact

Questions and feedbacks [welcome](mailto:contact@colinfay.me) !
