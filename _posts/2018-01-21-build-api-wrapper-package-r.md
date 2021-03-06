---
title: "[How to] Build an API wrapper package in 10 minutes."
author: colin_fay
post_date: 2018-02-04
layout: single
permalink: /build-api-wrapper-package-r/
categories: r-blog-en
output: jekyllthat::jekylldown
excerpt_separator: <!--more-->
---

Well… documentation not included (of course).

<!--more-->

API are cool. They allow to retrieve data from the web, and if ever
you’re familiar with {httr}, {jsonlite} and packages like these,
you’re able to build requests and retrieve data in a matter of
minutes.

But no worry, if you’re not familiar with http requests and web formats
like html, JSON and such, you can still go and look for a package
wrapper around that specific API: someone had probably been coding it
already.

And, if you want to be that someone that code an API wrapper package,
here’s a short tutorial that will allow you to create it.

> Disclaimer: the 10 minutes workflow does not (of course) include the
> package documentation.

## Find the API

Well, this first step can take a while… but, let’s say we have already
found it, and want to build an package around the french database for
addresses: <https://adresse.data.gouv.fr/api/>.

## Step 0: be sure you have all the packages you’ll need

Run this if you want to be sure you have all the packages we’ll use
here:

``` r
install.packages("devtools")
install.packages("roxygen2")
install.packages("usethis")
install.packages("curl")
install.packages("httr")
install.packages("jsonlite")
install.packages("attempt")
install.packages("purrr")
devtools::install_github("r-lib/desc")
```

## Step 1: the project

Create a new project with RStudio, and click on “Create a package with
devtools”.

## Step 2: devstuffs

  - In your terminal, run `usethis::use_data_raw()`.

  - Open a new R Script, save it into `/data_raw` as “devstuffs”, or any
    other name. Copy and paste into this script:

<!-- end list -->

``` r
library(devtools)
library(usethis)
library(desc)

# Remove default DESC
unlink("DESCRIPTION")
# Create and clean desc
my_desc <- description$new("!new")

# Set your package name
my_desc$set("Package", "yourpackage")

#Set your name
my_desc$set("Authors@R", "person('Colin', 'Fay', email = 'contact@colinfay.me', role = c('cre', 'aut'))")

# Remove some author fields
my_desc$del("Maintainer")

# Set the version
my_desc$set_version("0.0.0.9000")

# The title of your package
my_desc$set(Title = "My Supper API Wrapper")
# The description of your package
my_desc$set(Description = "A long description of this super package I'm working on.")
# The urls
my_desc$set("URL", "http://this")
my_desc$set("BugReports", "http://that")
# Save everyting
my_desc$write(file = "DESCRIPTION")

# If you want to use the MIT licence, code of conduct, and lifecycle badge
use_mit_license(name = "Colin FAY")
use_code_of_conduct()
use_lifecycle_badge("Experimental")
use_news_md()

# Get the dependencies
use_package("httr")
use_package("jsonlite")
use_package("curl")
use_package("attempt")
use_package("purrr")

# Clean your description
use_tidy_description()
```

Then, run everything from this script.

Copy and paste at the top of you README.Rmd
:

``` 
  [![lifecycle](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental)
```

And at the bottom
:

``` 
  Please note that this project is released with a [Contributor Code of Conduct](CODE_OF_CONDUCT.md).
  By participating in this project you agree to abide by its terms.
```

## Step 3: get the API url

There are two ways you can request on an API :

  - By building url: `url.and/path/to/the/data`

  - With parameters: `url.that/q=this&data=that`

This is the case for <https://adresse.data.gouv.fr/api/>. We’ve got the
base url: `http 'https://api-adresse.data.gouv.fr/search/?q=8 bd du
port'`, and search parameters.

Here, the base url is everything before the `?`, and the parameters are
the key-value pairs after the
.

![](https://github.com/ColinFay/colinfay.github.io/raw/master/_posts/img/api.png)

## Step 4: utils

Before creating the main calling functions, we’ll create some utilitary
functions that will run some tests: is the internet connexion running?
Does the {httr} result return the right http code?

For this, we’ll create a file called `utils.R`, save it in the R/
folder, and put into it:

``` r
#' @importFrom attempt stop_if_not
#' @importFrom curl has_internet
check_internet <- function(){
  stop_if_not(.x = has_internet(), msg = "Please check your internet connexion")
}

#' @importFrom httr status_code
check_status <- function(res){
  stop_if_not(.x = status_code(res), 
              .p = ~ .x == 200,
              msg = "The API returned an error")
}
```

In this same file we’ll also create two objects that will contain the
base API urls:

``` r
base_url <- "https://api-adresse.data.gouv.fr/search/"
reverse_url <- "https://api-adresse.data.gouv.fr/reverse/"
```

## Step 5 : the function that will call on the API

To call the API, we’ll use `GET` function from {httr}. Let’s first try
just this:

``` r
httr::GET(url = base_url, query = list(q = "Yeaye"))
```

    ## Response [https://api-adresse.data.gouv.fr/search/?q=Yeaye]
    ##   Date: 2018-02-04 21:40
    ##   Status: 200
    ##   Content-Type: application/json; charset=utf-8
    ##   Size: 574 B

As you can see, the status is 200. Which is a good sign: no error from
the API.

In the API we have chosen, there are 4 entry points: search, reverse,
and their counterparts with csv. These counterparts work by POSTing a
csv to the API, so let’s forget them for now.

The search entrypoint can take 8 parameters:

  - `q`: text search
  - `limit`: maximum number of results to return
  - `autocomplete`: autocompletion
  - `lat`: latitude
  - `lon`: longitude
  - `type`: type of elements to return (housenumber, street,place,
    municipality)
  - `postcode`: Postal code
  - `citycode`: INSEE code

These are the elements which will be passed as a list in the `query`
parameter from `httr::GET`.

> Note: if you’re going for an url based request
> (`url.and/path/to/the/data` or `url.and/?path=this&to=that`), you
> don’t need to set the query parameter, you can simply paste the url
> with the elements (`url <- paste0("url.and/?path=", this, "&to=",
> that)`).

Create a new R script and write the functions used to call the API:

``` r
#' Search the BAN
#' 
#' @param q text search
#' @param limit maximum number of results to return 
#' @param autocomplete autocompletion
#' @param lat latitude
#' @param lon longitude
#' @param type type of elements to return (housenumber, street,place, municipality)
#' @param postcode Postal code
#' @param citycode INSEE code
#'
#' @importFrom attempt stop_if_all
#' @importFrom purrr compact
#' @importFrom jsonlite fromJSON
#' @importFrom httr GET
#' @export
#' @rdname searchban
#'
#' @return the results from the search
#' @examples 
#' \dontrun{
#' search_ban("Rennes")
#' reverse_search_ban("48.11", "-1.68")
#' }

search_ban <- function(q = NULL, limit = NULL, autocomplete = NULL, lat = NULL, lon = NULL, type = NULL, postcode = NULL, citycode = NULL){
  args <- list(q = q, limit = limit, autocomplete = autocomplete, lat = lat, 
               lon = lon, type = type, postcode = postcode, citycode = citycode)
  # Check that at least one argument is not null
  stop_if_all(args, is.null, "You need to specify at least one argument")
  # Chek for internet
  check_internet()
  # Create the 
  res <- GET(base_url, query = compact(args))
  # Check the result
  check_status(res)
  # Get the content and return it as a data.frame
  fromJSON(rawToChar(res$content))$features
}

#' @export
#' @rdname searchban
reverse_search_ban <- function(lat = NULL, lon = NULL){
  args <- list(lat = lat, lon = lon)
  # Check that at least one argument is not null
  stop_if_all(args, is.null, "You need to specify at least one argument")
  # Chek for internet
  check_internet()
  # Create the 
  res <- GET(reverse_url, query = compact(args))
  # Check the result
  check_status(res)
  # Get the content and return it as a data.frame
  fromJSON(rawToChar(res$content))$features
}
```

> Note: you’ll need to change the arguments and documentation for your
> specific API (obviously).

If ever you want to a part of this roxygen filling programmatically, you
should check the excellent [{sinew}
package](https://github.com/metrumresearchgroup/sinew) by Jonathan Sidi.

## Step 6 : Roxygenise

Now, run in your console :

``` r
roxygen2::roxygenise()
```

And that’s it\! You’ve got a working package :)

## Step 7 : build your package

You can test that everything is ok with:

``` r
devtools::check()
# Then build it with:
devtools::build()
```

## What’s left

As I said, the 10 minutes workflow does not include the doc, so here’s
what left for you to do:

  - Write the Readme
  - Write a Vignette
  - Write tests

For that, go back to your dev file, add, and run:

``` r
use_testthat()
use_vignette("{your-package-name}")
use_readme_rmd()
```

And now, grab you best pen and write documentation ;)
