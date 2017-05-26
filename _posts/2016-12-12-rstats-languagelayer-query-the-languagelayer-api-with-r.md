---
ID: 1254
post_title: '#RStats — languagelayeR : query the languagelayer API with R'
author: colin_fay
post_date: 2016-12-12 23:07:28
post_excerpt: ""
layout: single
permalink: /rstats-languagelayer-query-the-languagelayer-api-with-r/
published: true
---
<h2>Improve your text analysis with this R package designed to access the languagelayer API.<!--more--></h2>
<p class="unchanged rich-diff-level-one">LanguagelayeR is now on <a href="https://cran.r-project.org/package=languagelayeR">CRAN</a></p>

<h3>languagelayerR</h3>
<p class="unchanged rich-diff-level-one">This package is designed to detect a language from a character string in R by acessing the languagelayer API — <a href="https://languagelayer.com/">https://languagelayer.com/</a></p>

<h2>Language layer API</h2>
<p class="unchanged rich-diff-level-one">This package offers a language detection tool by connecting to the languagelayer API, a JSON interface designed to extract language information from a character string.</p>

<h2>Install languagelayerR</h2>
<p">Install this package directly in R :</p>

<pre>devtools</span><span class="pl-k">::</span>install_github(<span class="pl-s"><span class="pl-pds">"</span>ColinFay/languagelayerR<span class="pl-pds">"</span></span>)</pre>
<h2>How languagelayeR works</h2>
<p class="unchanged rich-diff-level-one">The version 1.0.0 works with three functions. Which are :</p>

<ul class="unchanged rich-diff-level-one">
 	<li class="unchanged">
<p class="unchanged"><code>getLanguage</code> Get language information from a character string</p>
</li>
 	<li class="unchanged">
<p class="unchanged"><code>getSupportedLanguage</code> Get all current accessible languages on the languagelayer API</p>
</li>
 	<li class="unchanged">
<p class="unchanged"><code>setApiKey</code> Set your API key to access the languagelayer API</p>
</li>
</ul>
<h2>First of all</h2>
<p class="unchanged rich-diff-level-one">Before any request on the languagelayer, you need to set your API key for your current session. Use the function <code>setApiKey(apikey = "yourapikey")</code>.</p>
<p class="unchanged rich-diff-level-one">You can get your api key on your languagelayer <a href="https://languagelayer.com/dashboard">dashboard</a>.</p>

<h2>Examples</h2>
<h3>getLanguage</h3>
<p>Detect a language from a character string.</p>

<pre>getLanguage(<span class="pl-v">query</span> <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>I really really love R and that's a good thing, right?<span class="pl-pds">"</span></span>)</pre>
<h3 class="unchanged rich-diff-level-one">getSupportedLanguage</h3>
<p class="unchanged rich-diff-level-one">List all the languages available on the languagelayer API.</p>

<pre>getSupportedLanguage()</pre>
<h3>Contact</h3>
<p>Questions and feedbacks <a href="mailto:contact@colinfay.me">welcome</a> !</p>
&nbsp;
