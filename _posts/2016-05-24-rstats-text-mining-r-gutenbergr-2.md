---
ID: 1263
post_title: '#RStats — Text mining with R and gutenbergr'
author: colin_fay
post_date: 2016-05-24 18:00:32
post_excerpt: ""
layout: single
permalink: /rstats-text-mining-r-gutenbergr-2/
published: true
---
<div id="a-la-fois-technique-de-data-mining-et-graal-des-temps-modernes-la-fouille-de-texte-permet-de-faire-emerger-des-informations-depuis-une-large-quantite-de-texte.-comment-le-realiser-avec-r" class="section level2">
<h2>Introduction to text-mining with R and gutenbergr.</h2>
<!--more-->
<div id="le-text-mining-quest-ce-que-cest" class="section level3">
<h3>What is text-mining ?</h3>
At the crossroads of <strong>linguistics</strong>, <strong>computer science</strong> and <strong>statistics</strong>, text-mining is a data-mining technic used to analyze a corpus, in order to discover <strong>patterns</strong>, <strong>trends</strong> and <strong>singularities</strong> in a large number of texts. For example, you can analyse the Twitter description of your followers, or even get information from 5000 Facebook statuses, etc. Pretty cool, right?

The first "step" is perhaps the simplest to understand : <strong>frequency analysis</strong>. As the name suggests, this technique calculates the recurrence of each word inside a corpus — in other words... their frequency. This allows you to compare several texts. As an example, let's assume you're analysing 2500 comments on the Facebook page of your favorite brand / city / star, and find that among the most frequent words are "thank you", "beautiful", "super". If you take two competing brands / cities / stars, you come across "yeurk" and "hate" on one, and "disgusting" and "catastrophic" on the other... pretty straightforward, isn't it?

</div>
<h3>gutenbergr</h3>
I've chosen to analyse Lewis Caroll's famous masterpiece, <em>Alice's Adventures in Wonderland</em>. Why? I could have selected the last 1500 Tweets containing #Rennes ... but:
<ul>
 	<li>That has been done before</li>
 	<li>Good literature has never hurt anyone :)</li>
</ul>
<a href="https://cran.r-project.org/web/packages/gutenbergr/index.html">gutenbergr</a> is an R package you can use to dowload books from the <a href="https://www.gutenberg.org/">Gutenberg Project</a>.
<pre class="r"><code>library(gutenbergr)</code></pre>
<pre class="r"><code>aliceref &lt;- gutenberg_works(title == "Alice's Adventures in Wonderland")</code></pre>
This function gives you a list with the following elements:
<pre><code>## [1] "gutenberg_id"        "title"               "author"             
## [4] "gutenberg_author_id" "language"            "gutenberg_bookshelf"
## [7] "rights"              "has_text"</code></pre>
The first column contains the reference of the book you're looking for in the Gutenberg catalogue. You need this number to download the book:
<pre class="r"><code>library(magrittr)
alice &lt;- gutenberg_download(aliceref$gutenberg_id) %&gt;% gutenberg_strip()</code></pre>
Here, <code>gutenberg_download</code> takes the ID of the book you want to download, and returns you a data.frame with the full text. <code>gutenberg_strip</code> removes all the metadata at the beginning of the book.

</div>
<h3>Alice’s Adventures in Wonderland</h3>
<pre class="r"><code>library(tidytext)</code></pre>
To perform your data analysis, you'll need the <code>tidytext</code> package. Then :
<code> </code>
<pre class="r"><code>tidytext &lt;- data_frame(line = 1:nrow(alice), text = alice$text) %&gt;%
 unnest_tokens(word, text) %&gt;%
 anti_join(stop_words) %&gt;%
 count(word, sort = TRUE)
barplot(height=head(tidytext,10)$n, names.arg=head(tidytext,10)$word, xlab="Mots", ylab="Fréquence", col="#973232", main="Alice in Wonderland")

</code></pre>
Perfect! So…<em>drum rolls</em>…

<a href="https://colinfay.github.io/wp-content/uploads/2016/05/alice-in-wonderland.png"><img class="aligncenter size-full wp-image-1663" src="https://colinfay.github.io/wp-content/uploads/2016/05/alice-in-wonderland.png" alt="" width="1200" height="600" /></a>

Further reading (in french) :

<a href="http://data-bzh.fr/text-mining-r-part-2/">Racinisation et lemmatisation avec R</a>

<a href="http://data-bzh.fr/text-mining-r-part-3/">Créer un nuage de mots avec R</a>
<h3>Read more :</h3>
<a href="https://www.amazon.fr/gp/product/1491981652/ref=as_li_tl?ie=UTF8&amp;camp=1642&amp;creative=6746&amp;creativeASIN=1491981652&amp;linkCode=as2&amp;tag=dabz-21" rel="nofollow">Text Mining With R: A Tidy Approach</a><img style="border: none !important; margin: 0px !important;" src="http://ir-fr.amazon-adsystem.com/e/ir?t=dabz-21&amp;l=as2&amp;o=8&amp;a=1491981652" alt="" width="1" height="1" border="0" />
<a href="https://www.amazon.fr/gp/product/148336934X/ref=as_li_tl?ie=UTF8&amp;camp=1642&amp;creative=6746&amp;creativeASIN=148336934X&amp;linkCode=as2&amp;tag=dabz-21" rel="nofollow">Text Mining: A Guidebook for the Social Sciences</a><img style="border: none !important; margin: 0px !important;" src="http://ir-fr.amazon-adsystem.com/e/ir?t=dabz-21&amp;l=as2&amp;o=8&amp;a=148336934X" alt="" width="1" height="1" border="0" />
<a href="https://www.amazon.fr/gp/product/1461432227/ref=as_li_tl?ie=UTF8&amp;camp=1642&amp;creative=6746&amp;creativeASIN=1461432227&amp;linkCode=as2&amp;tag=dabz-21" rel="nofollow">Mining Text Data</a><img style="border: none !important; margin: 0px !important;" src="http://ir-fr.amazon-adsystem.com/e/ir?t=dabz-21&amp;l=as2&amp;o=8&amp;a=1461432227" alt="" width="1" height="1" border="0" />
<a href="https://www.amazon.fr/gp/product/3330006455/ref=as_li_tl?ie=UTF8&amp;camp=1642&amp;creative=6746&amp;creativeASIN=3330006455&amp;linkCode=as2&amp;tag=dabz-21" rel="nofollow">Text Mining</a><img style="border: none !important; margin: 0px !important;" src="http://ir-fr.amazon-adsystem.com/e/ir?t=dabz-21&amp;l=as2&amp;o=8&amp;a=3330006455" alt="" width="1" height="1" border="0" />
<a href="https://www.amazon.fr/gp/product/178355181X/ref=as_li_tl?ie=UTF8&amp;camp=1642&amp;creative=6746&amp;creativeASIN=178355181X&amp;linkCode=as2&amp;tag=dabz-21" rel="nofollow">Mastering Text Mining with R</a><img style="border: none !important; margin: 0px !important;" src="http://ir-fr.amazon-adsystem.com/e/ir?t=dabz-21&amp;l=as2&amp;o=8&amp;a=178355181X" alt="" width="1" height="1" border="0" />
<a href="https://www.amazon.fr/gp/product/1119282012/ref=as_li_tl?ie=UTF8&amp;camp=1642&amp;creative=6746&amp;creativeASIN=1119282012&amp;linkCode=as2&amp;tag=dabz-21" rel="nofollow">Text Mining in Practice With R</a><img style="border: none !important; margin: 0px !important;" src="http://ir-fr.amazon-adsystem.com/e/ir?t=dabz-21&amp;l=as2&amp;o=8&amp;a=1119282012" alt="" width="1" height="1" border="0" />
<a href="https://www.amazon.fr/gp/product/B00RZK7UCE/ref=as_li_tl?ie=UTF8&amp;camp=1642&amp;creative=6746&amp;creativeASIN=B00RZK7UCE&amp;linkCode=as2&amp;tag=dabz-21" rel="nofollow">Text Mining: From Ontology Learning to Automated Text Processing Applications</a><img style="border: none !important; margin: 0px !important;" src="http://ir-fr.amazon-adsystem.com/e/ir?t=dabz-21&amp;l=as2&amp;o=8&amp;a=B00RZK7UCE" alt="" width="1" height="1" border="0" />
<a href="https://www.amazon.fr/gp/product/B008KZULQ0/ref=as_li_tl?ie=UTF8&amp;camp=1642&amp;creative=6746&amp;creativeASIN=B008KZULQ0&amp;linkCode=as2&amp;tag=dabz-21" rel="nofollow">Text Mining: Classification, Clustering, and Applications</a><img style="border: none !important; margin: 0px !important;" src="http://ir-fr.amazon-adsystem.com/e/ir?t=dabz-21&amp;l=as2&amp;o=8&amp;a=B008KZULQ0" alt="" width="1" height="1" border="0" />
<a href="https://www.amazon.fr/gp/product/1627058982/ref=as_li_tl?ie=UTF8&amp;camp=1642&amp;creative=6746&amp;creativeASIN=1627058982&amp;linkCode=as2&amp;tag=dabz-21" rel="nofollow">Phrase Mining from Massive Text and Its Applications</a><img style="border: none !important; margin: 0px !important;" src="http://ir-fr.amazon-adsystem.com/e/ir?t=dabz-21&amp;l=as2&amp;o=8&amp;a=1627058982" alt="" width="1" height="1" border="0" />
<a href="https://www.amazon.fr/gp/product/B005UQLIA0/ref=as_li_tl?ie=UTF8&amp;camp=1642&amp;creative=6746&amp;creativeASIN=B005UQLIA0&amp;linkCode=as2&amp;tag=dabz-21" rel="nofollow">Text Mining: Applications and Theory</a><img style="border: none !important; margin: 0px !important;" src="http://ir-fr.amazon-adsystem.com/e/ir?t=dabz-21&amp;l=as2&amp;o=8&amp;a=B005UQLIA0" alt="" width="1" height="1" border="0" />
