---
ID: 1262
post_title: 'rpinterest : acess the Pinterest API with R'
author: colin_fay
post_date: 2016-07-30 20:17:45
post_excerpt: ""
layout: post
permalink: /rpinterest-package-r/
published: true
---
<h2>Access the Pinterest API with R with rpinterest. <!--more--></h2>
This package requests information from the Pinterest API.

rpinterest is now on <a href="https://cran.r-project.org/web/packages/rpinterest/index.html">CRAN</a>
<h2>Access the API</h2>
In order to get information from the API, you first need to get an access token from the <a href="https://developers.pinterest.com/tools/access_token/">Pinterest token generator</a>.
<h2>Install rpinterest</h2>
Install this package directly in R :
<div class="highlight highlight-source-r">
<pre><span class="pl-e">devtools</span><span class="pl-k">::</span>install_github(<span class="pl-s"><span class="pl-pds">"</span>ColinFay/rpinterest<span class="pl-pds">"</span></span>)</pre>
</div>
<h2>How rpinterest works</h2>
The version 0.1.0 works with seven functions. Which are :
<ul>
 	<li><code>BoardPinsByID</code> Get information about all the pins on a pinterest board using the board ID.</li>
 	<li><code>BoardPinsByName</code> Get information about all the pins on a pinterest board using the board name.</li>
 	<li><code>BoardSpecByID</code> Get information about a pinterest board using the board ID.</li>
 	<li><code>BoardSpecByName</code> Get information about a pinterest board using the board name.</li>
 	<li><code>PinSpecByID</code> Get information about a pinterest pin using the pin ID.</li>
 	<li><code>UserSpecByID</code> Get information about a pinterest user using the user ID.</li>
 	<li><code>UserSpecNyName</code> Get information about a pinterest user using the user name.</li>
</ul>
<h3>Contact</h3>
Questions and feedbacks <a href="mailto:contact@colinfay.me">welcome</a> !