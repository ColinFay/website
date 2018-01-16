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
    ##   '/private/var/folders/lz/thnnmbpd1rz0h1tmyzgg0mh00000gn/T/Rtmpmj9qAx/devtools6dbc68cd073c/rtweet'  \
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
rtweets04 <- search_tweets("#RStats", n = 10)
glimpse(rtweets04)
```

    ## Observations: 78
    ## Variables: 35
    ## $ screen_name                    <chr> "itknowingness", "Rbloggers", "...
    ## $ user_id                        <chr> "213339721", "144592995", "2831...
    ## $ created_at                     <dttm> 2018-01-16 10:40:02, 2018-01-1...
    ## $ status_id                      <chr> "953215261238317057", "95321525...
    ## $ text                           <chr> "RT @Rbloggers: Field Guide to ...
    ## $ retweet_count                  <int> 1, 1, 0, 0, 1, 1, 19, 1, 1, 27,...
    ## $ favorite_count                 <int> 0, 2, 0, 0, 0, 0, 0, 0, 0, 0, 1...
    ## $ is_quote_status                <lgl> FALSE, FALSE, FALSE, FALSE, FAL...
    ## $ quote_status_id                <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ is_retweet                     <lgl> TRUE, FALSE, FALSE, FALSE, TRUE...
    ## $ retweet_status_id              <chr> "953215255928360962", NA, NA, N...
    ## $ in_reply_to_status_status_id   <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ in_reply_to_status_user_id     <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ in_reply_to_status_screen_name <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ lang                           <chr> "en", "en", "en", "en", "en", "...
    ## $ source                         <chr> "ttools it knowingness", "r-blo...
    ## $ media_id                       <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ media_url                      <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ media_url_expanded             <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ urls                           <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ urls_display                   <chr> "wp.me/pMm6L-F6I", "wp.me/pMm6L...
    ## $ urls_expanded                  <chr> "https://wp.me/pMm6L-F6I", "htt...
    ## $ mentions_screen_name           <chr> "Rbloggers", NA, NA, "petermacp...
    ## $ mentions_user_id               <chr> "144592995", NA, NA, "916665295...
    ## $ symbols                        <lgl> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ hashtags                       <chr> "rstats DataScience", "rstats D...
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
rtweets04 %>% 
  unnest_tokens(word, text) %>% 
  select(screen_name, word) %>%
  slice(1:5)
```

    ## # A tibble: 5 x 2
    ##     screen_name      word
    ##           <chr>     <chr>
    ## 1 itknowingness        rt
    ## 2 itknowingness rbloggers
    ## 3 itknowingness     field
    ## 4 itknowingness     guide
    ## 5 itknowingness        to

### {rtweet} 0.6

The issue occurs when we try to do the exact same thing with the new
{rtweet}. Let’s start by updating and checking we have the latest
version.

``` r
detach("package:rtweet", unload=TRUE)
install.packages("rtweet", repos = "http://cran.us.r-project.org")
```

    ## 
    ## The downloaded binary packages are in
    ##  /var/folders/lz/thnnmbpd1rz0h1tmyzgg0mh00000gn/T//Rtmpmj9qAx/downloaded_packages

``` r
packageVersion("rtweet")
```

    ## [1] '0.6.0'

So if I try to do the exact same thing:

``` r
library(rtweet)

rtweets06 <- search_tweets("#RStats", n = 10)
```

    ## Searching for tweets...

    ## Finished collecting tweets!

``` r
glimpse(rtweets06)
```

    ## Observations: 8
    ## Variables: 42
    ## $ status_id              <chr> "953215261238317057", "9532152559283609...
    ## $ created_at             <dttm> 2018-01-16 10:40:02, 2018-01-16 10:40:...
    ## $ user_id                <chr> "213339721", "144592995", "2831401824",...
    ## $ screen_name            <chr> "itknowingness", "Rbloggers", "claczny"...
    ## $ text                   <chr> "RT @Rbloggers: Field Guide to the R Ec...
    ## $ source                 <chr> "ttools it knowingness", "r-bloggers.co...
    ## $ reply_to_status_id     <lgl> NA, NA, NA, NA, NA, NA, NA, NA
    ## $ reply_to_user_id       <lgl> NA, NA, NA, NA, NA, NA, NA, NA
    ## $ reply_to_screen_name   <lgl> NA, NA, NA, NA, NA, NA, NA, NA
    ## $ is_quote               <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALS...
    ## $ is_retweet             <lgl> TRUE, FALSE, FALSE, FALSE, TRUE, FALSE,...
    ## $ favorite_count         <int> 0, 2, 0, 0, 0, 0, 0, 0
    ## $ retweet_count          <int> 1, 1, 0, 0, 1, 1, 19, 1
    ## $ hashtags               <list> [<"rstats", "DataScience">, <"rstats",...
    ## $ symbols                <list> [NA, NA, NA, NA, NA, NA, NA, NA]
    ## $ urls_url               <list> ["wp.me/pMm6L-F6I", "wp.me/pMm6L-F6I",...
    ## $ urls_t.co              <list> ["https://t.co/4qcZwkhGHl", "https://t...
    ## $ urls_expanded_url      <list> ["https://wp.me/pMm6L-F6I", "https://w...
    ## $ media_url              <list> [NA, NA, NA, NA, NA, "http://pbs.twimg...
    ## $ media_t.co             <list> [NA, NA, NA, NA, NA, "https://t.co/RA0...
    ## $ media_expanded_url     <list> [NA, NA, NA, NA, NA, "https://twitter....
    ## $ media_type             <list> [NA, NA, NA, NA, NA, "photo", NA, NA]
    ## $ ext_media_url          <list> [NA, NA, NA, NA, NA, "http://pbs.twimg...
    ## $ ext_media_t.co         <list> [NA, NA, NA, NA, NA, "https://t.co/RA0...
    ## $ ext_media_expanded_url <list> [NA, NA, NA, NA, NA, "https://twitter....
    ## $ ext_media_type         <lgl> NA, NA, NA, NA, NA, NA, NA, NA
    ## $ mentions_user_id       <list> ["144592995", NA, NA, <"916665295", "9...
    ## $ mentions_screen_name   <list> ["Rbloggers", NA, NA, <"petermacp", "d...
    ## $ lang                   <chr> "en", "en", "en", "en", "en", "en", "en...
    ## $ quoted_status_id       <chr> NA, NA, NA, NA, NA, NA, NA, NA
    ## $ quoted_text            <chr> NA, NA, NA, NA, NA, NA, NA, NA
    ## $ retweet_status_id      <chr> "953215255928360962", NA, NA, NA, "9532...
    ## $ retweet_text           <chr> "Field Guide to the R Ecosystem https:/...
    ## $ place_url              <chr> NA, NA, NA, NA, NA, NA, NA, NA
    ## $ place_name             <chr> NA, NA, NA, NA, NA, NA, NA, NA
    ## $ place_full_name        <chr> NA, NA, NA, NA, NA, NA, NA, NA
    ## $ place_type             <chr> NA, NA, NA, NA, NA, NA, NA, NA
    ## $ country                <chr> NA, NA, NA, NA, NA, NA, NA, NA
    ## $ country_code           <chr> NA, NA, NA, NA, NA, NA, NA, NA
    ## $ geo_coords             <list> [<NA, NA>, <NA, NA>, <NA, NA>, <NA, NA...
    ## $ coords_coords          <list> [<NA, NA>, <NA, NA>, <NA, NA>, <NA, NA...
    ## $ bbox_coords            <list> [<NA, NA, NA, NA, NA, NA, NA, NA>, <NA...

``` r
rtweets06 %>% 
  unnest_tokens(word, text) %>% 
  select(screen_name, word) %>%
  slice(1:5)
```

    ## Error in unnest_tokens.data.frame(., word, text): unnest_tokens expects all columns of input to be atomic vectors (not lists)

This doesn’t work because {rtweet} results now have list columns, which
throws an error when we call `unnest_tokens()`.

## Workaround

For hashtags count :

``` r
library(purrr)
as_vector(rtweets06$hashtags) %>% 
   table() %>% 
   as.data.frame() %>% 
   arrange(Freq) %>% 
   top_n(5)
```

    ## Selecting by Freq

    ##                 . Freq
    ## 1    reproducible    1
    ## 2          Rstats    1
    ## 3    openresearch    2
    ## 4 reproducibility    2
    ## 5     DataScience    3
    ## 6          rstats    7

For mining the `text` column:

``` r
rtweets06 %>% 
  discard(is.list) %>%
  unnest_tokens(word, text) %>%
  count(word, sort = TRUE) %>% 
  top_n(5)
```

    ## Selecting by n

    ## # A tibble: 5 x 2
    ##     word     n
    ##    <chr> <int>
    ## 1 rstats     8
    ## 2     to     6
    ## 3  https     5
    ## 4   t.co     5
    ## 5    the     5

But the better workaround is to turn the list columns to a vector, by
combining the `modify_if` and the `simplify` functions from {purrr}:

``` r
rtweets06 %>% 
  modify_if(is.list, ~ simplify(.x) %>% paste(collapse = " "))  %>%
  unnest_tokens(word, text) %>%
  count(word, sort = TRUE) %>% 
  top_n(5)
```

    ## Selecting by n

    ## # A tibble: 5 x 2
    ##     word     n
    ##    <chr> <int>
    ## 1 rstats     8
    ## 2     to     6
    ## 3  https     5
    ## 4   t.co     5
    ## 5    the     5

``` r
rtweets06 %>% 
  modify_if(is.list, ~ simplify(.x) %>% paste(collapse = " ")) %>%
  unnest_tokens(word, hashtags) %>%
  count(word, sort = TRUE) %>% 
  top_n(5)
```

    ## Selecting by n

    ## # A tibble: 5 x 2
    ##              word     n
    ##             <chr> <int>
    ## 1          rstats    64
    ## 2     datascience    24
    ## 3    openresearch    16
    ## 4 reproducibility    16
    ## 5    reproducible     8
