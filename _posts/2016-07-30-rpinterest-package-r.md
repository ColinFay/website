---
title: "rpinterest : acess the Pinterest API with R"
author: colin_fay
post_date: 2016-07-30 20:17:45
layout: single
permalink: /rpinterest-package-r/
published: true
categories : r-blog-en
---
## Access the Pinterest API with R with rpinterest. <!--more-->
This package requests information from the Pinterest API.

rpinterest is now on <a href="https://cran.r-project.org/web/packages/rpinterest/index.html">CRAN</a>
## Access the API
In order to get information from the API, you first need to get an access token from the <a href="https://developers.pinterest.com/tools/access_token/">Pinterest token generator</a>.
## Install rpinterest
Install this package directly in R :
<div class="highlight highlight-source-r">
<pre><span class="pl-e">devtools<span class="pl-k">::install_github(<span class="pl-s"><span class="pl-pds">"ColinFay/rpinterest<span class="pl-pds">"){% endhighlight %}

## How rpinterest works
The version 0.1.0 works with seven functions. Which are :
<ul>
 	<li>{% highlight r %} 
BoardPinsByID
{% endhighlight %} Get information about all the pins on a pinterest board using the board ID.</li>
 	<li>{% highlight r %} 
BoardPinsByName
{% endhighlight %} Get information about all the pins on a pinterest board using the board name.</li>
 	<li>{% highlight r %} 
BoardSpecByID
{% endhighlight %} Get information about a pinterest board using the board ID.</li>
 	<li>{% highlight r %} 
BoardSpecByName
{% endhighlight %} Get information about a pinterest board using the board name.</li>
 	<li>{% highlight r %} 
PinSpecByID
{% endhighlight %} Get information about a pinterest pin using the pin ID.</li>
 	<li>{% highlight r %} 
UserSpecByID
{% endhighlight %} Get information about a pinterest user using the user ID.</li>
 	<li>{% highlight r %} 
UserSpecNyName
{% endhighlight %} Get information about a pinterest user using the user name.</li>
</ul>
### Contact
Questions and feedbacks <a href="mailto:contact@colinfay.me">welcome</a> !
