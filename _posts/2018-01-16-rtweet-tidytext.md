---
title: "Combining the new {rtweet} and {tidytext}"
author: colin_fay
post_date: 2018-01-16
layout: single
permalink: /rtweet-tidytext/
categories: r-blog-en
output: jekyllthat::jekylldown
excerpt_separator: <!--more-->
---

How to deal with {rwteet} v0.6.\* and {tidytext}. Spoiler: you’ll need
some {purrr}.

<!--more-->

Here’s a complementary post explaining why you can’t reproduce exactly
reproduce the code found in:

  - [Collecting tweets with R and
    {rtweet}](http://colinfay.me/collect-rtweet/) - (**This post has
    been updated**)

  - [Visualising text data with
    ggplot2](https://github.com/ColinFay/conf/blob/master/2017-11-budapest/fay_colin_text_data_ggplot2.pdf)

## New {rtweet} vs {tidytext}

Last update of {rtweet} (last published on CRAN on 2017-11-16, so after
I published the blogposts / slides I just mentioned) comes with a new
feature: list columns.

Problem is: you can’t pass to `tidytext::unnest_tokens` a data frame
containing list-columns. This is prevented by the third to fifth lines
of `tidytext:::unnest_tokens.data.frame`:

``` r
if (any(!purrr::map_lgl(tbl, is.atomic))) {
  stop("unnest_tokens expects all columns of input to be atomic vectors (not lists)")
}
```

### Getting back to {rtweet} 0.4

This was not an issue with the old {rtweet}, as it didn’t return a data
frame with list-columns.

To verify this, you can get the previous version of {rtweet} with the
`install_version()` function from {devtools}:

``` r
library(devtools)
install_version("rtweet", version = "0.4.0", repos = "http://cran.us.r-project.org")
```

    ## Downloading package from url: http://cran.us.r-project.org/src/contrib/Archive/rtweet/rtweet_0.4.0.tar.gz

    ## Installing rtweet

    ## '/Library/Frameworks/R.framework/Resources/bin/R' --no-site-file  \
    ##   --no-environ --no-save --no-restore --quiet CMD INSTALL  \
    ##   '/private/var/folders/lz/thnnmbpd1rz0h1tmyzgg0mh00000gn/T/RtmpGnlMdT/devtools6c4ac1db626/rtweet'  \
    ##   --library='/Library/Frameworks/R.framework/Versions/3.4/Resources/library'  \
    ##   --install-tests

    ## 

``` r
packageVersion("rtweet")
```

    ## [1] '0.4.0'

``` r
library(rtweet)
library(dplyr)
tweets <- search_tweets("#RStats", n = 10)
glimpse(tweets)
```

    ## Observations: 79
    ## Variables: 35
    ## $ screen_name                    <chr> "oxshef_dataviz", "martinjhnhad...
    ## $ user_id                        <chr> "936603311268139009", "52321651...
    ## $ created_at                     <dttm> 2018-01-16 10:34:39, 2018-01-1...
    ## $ status_id                      <chr> "953213903722438656", "95321376...
    ## $ text                           <chr> "RT @martinjhnhadley: #reproduc...
    ## $ retweet_count                  <int> 1, 1, 19, 1, 1, 27, 1, 0, 7, 1,...
    ## $ favorite_count                 <int> 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0...
    ## $ is_quote_status                <lgl> FALSE, FALSE, FALSE, FALSE, FAL...
    ## $ quote_status_id                <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ is_retweet                     <lgl> TRUE, FALSE, TRUE, FALSE, TRUE,...
    ## $ retweet_status_id              <chr> "953213760499601408", NA, "9530...
    ## $ in_reply_to_status_status_id   <chr> NA, NA, NA, NA, NA, NA, "952870...
    ## $ in_reply_to_status_user_id     <chr> NA, NA, NA, NA, NA, NA, "188816...
    ## $ in_reply_to_status_screen_name <chr> NA, NA, NA, NA, NA, NA, "betati...
    ## $ lang                           <chr> "en", "en", "en", "en", "en", "...
    ## $ source                         <chr> "Twitter for iPhone", "Twitter ...
    ## $ media_id                       <chr> NA, NA, NA, NA, NA, "9518197540...
    ## $ media_url                      <chr> NA, NA, NA, NA, NA, "http://pbs...
    ## $ media_url_expanded             <chr> NA, NA, NA, NA, NA, "https://tw...
    ## $ urls                           <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ urls_display                   <chr> NA, "twitter.com/i/web/status/9...
    ## $ urls_expanded                  <chr> NA, "https://twitter.com/i/web/...
    ## $ mentions_screen_name           <chr> "martinjhnhadley", NA, "Rblogge...
    ## $ mentions_user_id               <chr> "523216510", NA, "144592995", N...
    ## $ symbols                        <lgl> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ hashtags                       <chr> "reproducibility openresearch r...
    ## $ coordinates                    <lgl> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ place_id                       <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ place_type                     <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ place_name                     <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ place_full_name                <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ country_code                   <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ country                        <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ bounding_box_coordinates       <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ bounding_box_type              <chr> NA, NA, NA, NA, NA, NA, NA, NA,...

So, no problem for doing (as you can find in the slides):

``` r
library(tidytext)
tweets %>% 
  unnest_tokens(word, text) %>% 
  select(screen_name, word) %>%
  slice(1:5)
```

    ## # A tibble: 5 x 2
    ##      screen_name            word
    ##            <chr>           <chr>
    ## 1 oxshef_dataviz              rt
    ## 2 oxshef_dataviz martinjhnhadley
    ## 3 oxshef_dataviz reproducibility
    ## 4 oxshef_dataviz             and
    ## 5 oxshef_dataviz    openresearch

### {rtweet} 0.6

The issue occurs when we try to do the exact same thing with the new
{rtweet}. Let’s start by updating and checking we have the latest
version.

``` r
install.packages("rtweet", repos = "http://cran.us.r-project.org")
```

    ## 
    ## The downloaded binary packages are in
    ##  /var/folders/lz/thnnmbpd1rz0h1tmyzgg0mh00000gn/T//RtmpGnlMdT/downloaded_packages

``` r
packageVersion("rtweet")
```

    ## [1] '0.6.0'

So if I try to do the exact same thing:

``` r
library(rtweet)
library(dplyr)
library(tidytext)

tweets <- search_tweets("#RStats", n = 10)
```

    ## Searching for tweets...

    ## Finished collecting tweets!

``` r
glimpse(tweets)
```

    ## Observations: 79
    ## Variables: 35
    ## $ screen_name                    <chr> "oxshef_dataviz", "martinjhnhad...
    ## $ user_id                        <chr> "936603311268139009", "52321651...
    ## $ created_at                     <dttm> 2018-01-16 10:34:39, 2018-01-1...
    ## $ status_id                      <chr> "953213903722438656", "95321376...
    ## $ text                           <chr> "RT @martinjhnhadley: #reproduc...
    ## $ retweet_count                  <int> 1, 1, 19, 1, 1, 27, 1, 0, 7, 1,...
    ## $ favorite_count                 <int> 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0...
    ## $ is_quote_status                <lgl> FALSE, FALSE, FALSE, FALSE, FAL...
    ## $ quote_status_id                <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ is_retweet                     <lgl> TRUE, FALSE, TRUE, FALSE, TRUE,...
    ## $ retweet_status_id              <chr> "953213760499601408", NA, "9530...
    ## $ in_reply_to_status_status_id   <chr> NA, NA, NA, NA, NA, NA, "952870...
    ## $ in_reply_to_status_user_id     <chr> NA, NA, NA, NA, NA, NA, "188816...
    ## $ in_reply_to_status_screen_name <chr> NA, NA, NA, NA, NA, NA, "betati...
    ## $ lang                           <chr> "en", "en", "en", "en", "en", "...
    ## $ source                         <chr> "Twitter for iPhone", "Twitter ...
    ## $ media_id                       <chr> NA, NA, NA, NA, NA, "9518197540...
    ## $ media_url                      <chr> NA, NA, NA, NA, NA, "http://pbs...
    ## $ media_url_expanded             <chr> NA, NA, NA, NA, NA, "https://tw...
    ## $ urls                           <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ urls_display                   <chr> NA, "twitter.com/i/web/status/9...
    ## $ urls_expanded                  <chr> NA, "https://twitter.com/i/web/...
    ## $ mentions_screen_name           <chr> "martinjhnhadley", NA, "Rblogge...
    ## $ mentions_user_id               <chr> "523216510", NA, "144592995", N...
    ## $ symbols                        <lgl> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ hashtags                       <chr> "reproducibility openresearch r...
    ## $ coordinates                    <lgl> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ place_id                       <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ place_type                     <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ place_name                     <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ place_full_name                <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ country_code                   <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ country                        <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ bounding_box_coordinates       <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ bounding_box_type              <chr> NA, NA, NA, NA, NA, NA, NA, NA,...

``` r
tweets %>% 
  unnest_tokens(word, text) %>% 
  select(screen_name, word) %>%
  slice(1:5)
```

    ## # A tibble: 5 x 2
    ##      screen_name            word
    ##            <chr>           <chr>
    ## 1 oxshef_dataviz              rt
    ## 2 oxshef_dataviz martinjhnhadley
    ## 3 oxshef_dataviz reproducibility
    ## 4 oxshef_dataviz             and
    ## 5 oxshef_dataviz    openresearch

This doesn’t work because {rtweet} results now have list columns, which
throws an error when we call `unnest_tokens()`.

## Workaround

For hashtags count :

``` r
library(purrr)
as_vector(tweets$hashtags) %>% 
   table() %>% 
   as.data.frame() %>% 
   arrange(Freq) %>% 
   top_n(5)
```

    ## Selecting by Freq

    ##                                                     . Freq
    ## 1                                              RStats    4
    ## 2 Agile Design business Coding Rstats Java python IoT    6
    ## 3                                              Rstats    6
    ## 4                                  rstats DataScience    6
    ## 5                                              rstats   33

For mining the `text` column:

``` r
tweets %>% 
  discard(is.list) %>%
  unnest_tokens(word, text) %>%
  count(word, sort = TRUE) %>% 
  top_n(5)
```

    ## Selecting by n

    ## # A tibble: 5 x 2
    ##     word     n
    ##    <chr> <int>
    ## 1 rstats    77
    ## 2  https    65
    ## 3   t.co    63
    ## 4     rt    59
    ## 5    for    33

But the better workaround is to turn the list columns to a vector, by
combining the `modify_if` and the `simplify` functions from {purrr}:

``` r
tweets %>% 
  modify_if(is.list, ~ simplify(.x) %>% paste(collapse = " "))  %>%
  unnest_tokens(word, text) %>%
  count(word, sort = TRUE) %>% 
  top_n(5)
```

    ## Selecting by n

    ## # A tibble: 5 x 2
    ##     word     n
    ##    <chr> <int>
    ## 1 rstats    77
    ## 2  https    65
    ## 3   t.co    63
    ## 4     rt    59
    ## 5    for    33

``` r
tweets %>% 
  modify_if(is.list, ~ simplify(.x) %>% paste(collapse = " ")) %>%
  unnest_tokens(word, hashtags) %>%
  count(word, sort = TRUE) %>% 
  top_n(5)
```

    ## Selecting by n

    ## # A tibble: 7 x 2
    ##          word     n
    ##         <chr> <int>
    ## 1      rstats    77
    ## 2 datascience    12
    ## 3      python    12
    ## 4        java    10
    ## 5    business     9
    ## 6      coding     9
    ## 7         iot     9
