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

On the importance of keeping your packages up to date.

<!--more-->

I’ve received recently some mails and comments asking why the code in :

  - [Collecting tweets with R and
    {rtweet}](http://colinfay.me/collect-rtweet/)

  - [Visualising text data with
    ggplot2](https://github.com/ColinFay/conf/blob/master/2017-11-budapest/fay_colin_text_data_ggplot2.pdf)

couldn’t be reproduced.

Spoiler: this was due to the behavior of an old version of {tidytext}.
As the new version is out, this problem is solved. But I thought I could
use this as the perfect example to show how to go to previous version of
a package :)

## New {rtweet} vs previous {tidytext}

Last update of {rtweet} ( published on CRAN on 2017-11-16, so after I
published the blogposts / slides I just mentioned) comes with a new
feature: list columns.

Problem is: you can’t pass a data frame containing list-columns to
`tidytext::unnest_tokens`, in version 0.1.5 of tidytext. This was
prevented by the third to fifth lines of
`tidytext:::unnest_tokens.data.frame`:

``` r
if (any(!purrr::map_lgl(tbl, is.atomic))) {
  stop("unnest_tokens expects all columns of input to be atomic vectors (not lists)")
}
```

### Getting back to previous version of packages

There was no issue with the old {rtweet} and {tidytext}. To verify this,
let’s get back to previous versions of these two packages with the
`install_version()` function from {devtools}:

``` r
library(devtools)
install_version("rtweet", version = "0.4.0", repos = "http://cran.us.r-project.org", quiet = TRUE)
```

    ## Downloading package from url: http://cran.us.r-project.org/src/contrib/Archive/rtweet/rtweet_0.4.0.tar.gz

    ## Installing rtweet

    ## '/Library/Frameworks/R.framework/Resources/bin/R' --no-site-file  \
    ##   --no-environ --no-save --no-restore --quiet CMD INSTALL  \
    ##   '/private/var/folders/lz/thnnmbpd1rz0h1tmyzgg0mh00000gn/T/RtmpoT2sg4/devtools74663c0dc0e8/rtweet'  \
    ##   --library='/Library/Frameworks/R.framework/Versions/3.4/Resources/library'  \
    ##   --install-tests

    ## 

``` r
install_version("tidytext", version = "0.1.5", repos = "http://cran.us.r-project.org", quiet = TRUE)
```

    ## Downloading package from url: http://cran.us.r-project.org/src/contrib/Archive/tidytext/tidytext_0.1.5.tar.gz

    ## Installing tidytext

    ## '/Library/Frameworks/R.framework/Resources/bin/R' --no-site-file  \
    ##   --no-environ --no-save --no-restore --quiet CMD INSTALL  \
    ##   '/private/var/folders/lz/thnnmbpd1rz0h1tmyzgg0mh00000gn/T/RtmpoT2sg4/devtools7466486fa8/tidytext'  \
    ##   --library='/Library/Frameworks/R.framework/Versions/3.4/Resources/library'  \
    ##   --install-tests

    ## 

``` r
packageVersion("rtweet")
```

    ## [1] '0.4.0'

``` r
packageVersion("tidytext")
```

    ## [1] '0.1.5'

``` r
library(rtweet)
library(dplyr)
library(tidytext)
rtweets04 <- search_tweets("#RStats", n = 10)
glimpse(rtweets04)
```

    ## Observations: 77
    ## Variables: 35
    ## $ screen_name                    <chr> "rstatsbot1234", "rstatsbot1234...
    ## $ user_id                        <chr> "947752259038801920", "94775225...
    ## $ created_at                     <dttm> 2018-01-16 11:30:08, 2018-01-1...
    ## $ status_id                      <chr> "953227866375798784", "95322786...
    ## $ text                           <chr> "RT @mdsumner: guide to GPU-acc...
    ## $ retweet_count                  <int> 1, 1, 1, 2, 10, 1, 10, 5, 2, 20...
    ## $ favorite_count                 <int> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ is_quote_status                <lgl> FALSE, FALSE, FALSE, FALSE, FAL...
    ## $ quote_status_id                <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ is_retweet                     <lgl> TRUE, TRUE, TRUE, TRUE, TRUE, F...
    ## $ retweet_status_id              <chr> "953220953269399552", "95322156...
    ## $ in_reply_to_status_status_id   <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ in_reply_to_status_user_id     <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ in_reply_to_status_screen_name <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ lang                           <chr> "en", "en", "es", "en", "en", "...
    ## $ source                         <chr> "Rstats1234", "Rstats1234", "Rs...
    ## $ media_id                       <chr> NA, NA, NA, "952943350264479744...
    ## $ media_url                      <chr> NA, NA, NA, "http://pbs.twimg.c...
    ## $ media_url_expanded             <chr> NA, NA, NA, "https://twitter.co...
    ## $ urls                           <chr> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ urls_display                   <chr> "appsilondatascience.com/blog/r...
    ## $ urls_expanded                  <chr> "https://appsilondatascience.co...
    ## $ mentions_screen_name           <chr> "mdsumner", "_ColinFay WinVecto...
    ## $ mentions_user_id               <chr> "103516223", "84618490 13249607...
    ## $ symbols                        <lgl> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ hashtags                       <chr> "rstats", "RStats", "rstats", "...
    ## $ coordinates                    <lgl> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ place_id                       <lgl> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ place_type                     <lgl> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ place_name                     <lgl> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ place_full_name                <lgl> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ country_code                   <lgl> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ country                        <lgl> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ bounding_box_coordinates       <lgl> NA, NA, NA, NA, NA, NA, NA, NA,...
    ## $ bounding_box_type              <lgl> NA, NA, NA, NA, NA, NA, NA, NA,...

So, no problem for doing this (as you can find in the slides):

``` r
rtweets04 %>% 
  unnest_tokens(word, text) %>% 
  select(screen_name, word) %>%
  slice(1:5)
```

    ## # A tibble: 5 x 2
    ##     screen_name     word
    ##           <chr>    <chr>
    ## 1 rstatsbot1234       rt
    ## 2 rstatsbot1234 mdsumner
    ## 3 rstatsbot1234    guide
    ## 4 rstatsbot1234       to
    ## 5 rstatsbot1234      gpu

### {rtweet} 0.6

For a while, there were an issue while trying to use these two packages
({rtweet} 0.6 was released on 2017-11-16, and {tidytext} 0.1.6 on
2018-01-07).

Let’s simulate this behavior by updating to {rtweet} 0.6, while staying
at {tidytext} 0.1.5.

``` r
detach("package:rtweet", unload=TRUE)
install.packages("rtweet", repos = "http://cran.us.r-project.org", quiet = TRUE)
packageVersion("rtweet")
```

    ## [1] '0.6.0'

``` r
packageVersion("tidytext")
```

    ## [1] '0.1.5'

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

    ## Observations: 10
    ## Variables: 42
    ## $ status_id              <chr> "953227866375798784", "9532278621227089...
    ## $ created_at             <dttm> 2018-01-16 11:30:08, 2018-01-16 11:30:...
    ## $ user_id                <chr> "947752259038801920", "9477522590388019...
    ## $ screen_name            <chr> "rstatsbot1234", "rstatsbot1234", "rsta...
    ## $ text                   <chr> "RT @mdsumner: guide to GPU-accelerated...
    ## $ source                 <chr> "Rstats1234", "Rstats1234", "Rstats1234...
    ## $ reply_to_status_id     <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA
    ## $ reply_to_user_id       <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA
    ## $ reply_to_screen_name   <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA
    ## $ is_quote               <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALS...
    ## $ is_retweet             <lgl> TRUE, TRUE, TRUE, TRUE, TRUE, FALSE, TR...
    ## $ favorite_count         <int> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
    ## $ retweet_count          <int> 1, 1, 1, 2, 10, 1, 10, 5, 2, 20
    ## $ hashtags               <list> ["rstats", "RStats", "rstats", <"R", "...
    ## $ symbols                <list> [NA, NA, NA, NA, NA, NA, NA, NA, NA, NA]
    ## $ urls_url               <list> ["appsilondatascience.com/blog/rstats/...
    ## $ urls_t.co              <list> ["https://t.co/mpulM1UJjq", "https://t...
    ## $ urls_expanded_url      <list> ["https://appsilondatascience.com/blog...
    ## $ media_url              <list> [NA, NA, NA, "http://pbs.twimg.com/med...
    ## $ media_t.co             <list> [NA, NA, NA, "https://t.co/ChU84PAXSc"...
    ## $ media_expanded_url     <list> [NA, NA, NA, "https://twitter.com/Data...
    ## $ media_type             <list> [NA, NA, NA, "photo", NA, NA, NA, NA, ...
    ## $ ext_media_url          <list> [NA, NA, NA, "http://pbs.twimg.com/med...
    ## $ ext_media_t.co         <list> [NA, NA, NA, "https://t.co/ChU84PAXSc"...
    ## $ ext_media_expanded_url <list> [NA, NA, NA, "https://twitter.com/Data...
    ## $ ext_media_type         <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA
    ## $ mentions_user_id       <list> ["103516223", <"84618490", "1324960710...
    ## $ mentions_screen_name   <list> ["mdsumner", <"_ColinFay", "WinVectorL...
    ## $ lang                   <chr> "en", "en", "es", "en", "en", "en", "en...
    ## $ quoted_status_id       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA
    ## $ quoted_text            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA
    ## $ retweet_status_id      <chr> "953220953269399552", "9532215626767114...
    ## $ retweet_text           <chr> "guide to GPU-accelerated ship recognit...
    ## $ place_url              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA
    ## $ place_name             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA
    ## $ place_full_name        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA
    ## $ place_type             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA
    ## $ country                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA
    ## $ country_code           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA
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
throws an error when we call `unnest_tokens()` from {tidytext} 0.1.5.

But everything is now back in order with the new {tidytext} version :

``` r
detach("package:tidytext", unload=TRUE)
install.packages("tidytext", repos = "http://cran.us.r-project.org", quiet = TRUE)
```

    ## 
    ##   There is a binary version available but the source version is
    ##   later:
    ##          binary source needs_compilation
    ## tidytext  0.1.5  0.1.6             FALSE

    ## installing the source package 'tidytext'

``` r
packageVersion("tidytext")
```

    ## [1] '0.1.6'

``` r
rtweets06 %>% 
  tidytext::unnest_tokens(word, text) %>% 
  select(screen_name, word) %>%
  slice(1:5)
```

    # A tibble: 5 x 2
      screen_name      word
            <chr>     <chr>
    1   CheezeViz        rt
    2   CheezeViz rbloggers
    3   CheezeViz         a
    4   CheezeViz     guide
    5   CheezeViz        to

So, in conclusion: don’t forget to update your packages\!

## Workaround

Here’s a {purrr} workaround I used for a while, when the two versions
were not in sync. I’m just putting it there just for the sake of
sharing:

``` r
library(purrr)
rtweets06 %>%
   modify_if(is.list, ~ simplify(.x) %>% paste(collapse = " "))  %>%
   tidytext::unnest_tokens(word, text) %>%
  select(screen_name, word) %>%
  slice(1:5)
```

    # A tibble: 5 x 2
      screen_name      word
            <chr>     <chr>
    1   CheezeViz        rt
    2   CheezeViz rbloggers
    3   CheezeViz         a
    4   CheezeViz     guide
    5   CheezeViz        to
