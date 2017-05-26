---
ID: 1260
post_title: 'Data &#038; Vinyls — A discogs library exploration with R'
author: colin_fay
post_date: 2016-08-31 12:00:36
post_excerpt: ""
layout: single
permalink: /data-vinyles-discogs-r/
published: true
---
<h2>As a vinyl lover and data addict, I had some fun making requests on the Discogs API with R, in order to know better what is inside my library.</h2>
<!--more-->

Every vinyl lover knows about Discogs. But did you know you could easily access the API? Here are the lines of code I used to access my library.

<span style="text-decoration: underline;">Note</span> : You can download the data used here in <a href="http://colinfay.me/data/collection_complete.json" target="_blank">JSON</a>, or directly in R :
<pre class="r"><code>collection_complete &lt;- jsonlite::fromJSON(txt = "http://colinfay.me/data/collection_complete.json", simplifyDataFrame = TRUE)</code></pre>
<h3>Major Tom to Discogs API</h3>
Before starting, I'll need these two functions: <em><code>%&gt;% </code></em>and <em><code>%||%</code>.</em>
<pre class="r"><code>library(magrittr) #for %&gt;%
`%||%` &lt;- function(a,b) if(is.null(a)) b else a</code></pre>
Let's first get my Discogs profile:
<pre class="r"><code>user &lt;- "_colin"
content &lt;- httr::GET(paste0("https://api.discogs.com/users/", user, "/collection/folders"))
content &lt;- rjson::fromJSON(rawToChar(content$content))$folders
content</code></pre>
<pre><code>## [[1]]
## [[1]]$count
## [1] 308
## 
## [[1]]$resource_url
## [1] "https://api.discogs.com/users/_colin/collection/folders/0"
## 
## [[1]]$id
## [1] 0
## 
## [[1]]$name
## [1] "All"</code></pre>
This first request brings in the environment all the information about a profil (here "_colin", aka : me).

<code>$count</code> tells us the number of entries in the library : 308.  The <code>$id</code> element is the number of “folders” created by the user – 0 corresponding to the whole collection, without any list specification.
<h3>Create a dataframe with all the vinyls</h3>
The Discogs API sends back pages with 100 max results. Here, my collection has 308, so I'll use a <code>repeat</code> loop to query all the data, and store them in a dataframe.
<pre class="r"><code>collec_url &lt;- httr::GET(paste0("https://api.discogs.com/users/", user, "/collection/folders/", content[[1]]$id, "/releases?page=1&amp;per_page=100"))
if (collec_url$status_code == 200){
  collec &lt;- rjson::fromJSON(rawToChar(collec_url$content))
  collecdata &lt;- collec$releases
    if(!is.null(collec$pagination$urls$`next`)){
      repeat{
        url &lt;- httr::GET(collec$pagination$urls$`next`)
        collec &lt;- rjson::fromJSON(rawToChar(url$content))
        collecdata &lt;- c(collecdata, collec$releases)
        if(is.null(collec$pagination$urls$`next`)){
          break
        }
      }
    }
}
  collection &lt;- lapply(collecdata, function(obj){
    data.frame(release_id = obj$basic_information$id %||% NA,
               label = obj$basic_information$labels[[1]]$name %||% NA,
               year = obj$basic_information$year %||% NA,
               title = obj$basic_information$title %||% NA, 
               artist_name = obj$basic_information$artists[[1]]$name %||% NA,
               artist_id = obj$basic_information$artists[[1]]$id %||% NA,
               artist_resource_url = obj$basic_information$artists[[1]]$resource_url %||% NA, 
               format = obj$basic_information$formats[[1]]$name %||% NA,
               resource_url = obj$basic_information$resource_url %||% NA)
  }) %&gt;% do.call(rbind, .) %&gt;% 
    unique()
</code></pre>
Here is what the dataframe looks like:
<pre class="r"><code>library(pander)
pander(head(collection))</code></pre>
<table style="width: 96%;"><caption>Table continues below</caption><colgroup> <col width="18%" /> <col width="29%" /> <col width="9%" /> <col width="38%" /> </colgroup>
<thead>
<tr class="header">
<th align="center">release_id</th>
<th align="center">label</th>
<th align="center">year</th>
<th align="center">title</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">5181773</td>
<td align="center">A&amp;M Records</td>
<td align="center">1982</td>
<td align="center">Night And Day</td>
</tr>
<tr class="even">
<td align="center">3690646</td>
<td align="center">A&amp;M Records (2)</td>
<td align="center">2012</td>
<td align="center">God Save The Queen</td>
</tr>
<tr class="odd">
<td align="center">944917</td>
<td align="center">Alexi Delano Limited</td>
<td align="center">2007</td>
<td align="center">The Acid Sessions Vol. 4</td>
</tr>
<tr class="even">
<td align="center">906983</td>
<td align="center">Alphabet City</td>
<td align="center">2007</td>
<td align="center">Urban Minds / Skattered</td>
</tr>
<tr class="odd">
<td align="center">8112758</td>
<td align="center">Amerilys</td>
<td align="center">1986</td>
<td align="center">Follement Vôtre</td>
</tr>
<tr class="even">
<td align="center">5800664</td>
<td align="center">Anette Records</td>
<td align="center">2014</td>
<td align="center">And The Dead Shall Lie There</td>
</tr>
</tbody>
</table>
<table><caption>Table continues below</caption><colgroup> <col width="20%" /> <col width="16%" /> <col width="52%" /> <col width="10%" /> </colgroup>
<thead>
<tr class="header">
<th align="center">artist_name</th>
<th align="center">artist_id</th>
<th align="center">artist_resource_url</th>
<th align="center">format</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">Joe Jackson</td>
<td align="center">75280</td>
<td align="center"><a class="uri" href="https://api.discogs.com/artists/75280">https://api.discogs.com/artists/75280</a></td>
<td align="center">Vinyl</td>
</tr>
<tr class="even">
<td align="center">Sex Pistols</td>
<td align="center">31753</td>
<td align="center"><a class="uri" href="https://api.discogs.com/artists/31753">https://api.discogs.com/artists/31753</a></td>
<td align="center">Vinyl</td>
</tr>
<tr class="odd">
<td align="center">Alexi Delano</td>
<td align="center">26</td>
<td align="center"><a class="uri" href="https://api.discogs.com/artists/26">https://api.discogs.com/artists/26</a></td>
<td align="center">Vinyl</td>
</tr>
<tr class="even">
<td align="center">Pacjam</td>
<td align="center">488187</td>
<td align="center"><a class="uri" href="https://api.discogs.com/artists/488187">https://api.discogs.com/artists/488187</a></td>
<td align="center">Vinyl</td>
</tr>
<tr class="odd">
<td align="center">Diane Dufresne</td>
<td align="center">647100</td>
<td align="center"><a class="uri" href="https://api.discogs.com/artists/647100">https://api.discogs.com/artists/647100</a></td>
<td align="center">Vinyl</td>
</tr>
<tr class="even">
<td align="center">Ancient Mith</td>
<td align="center">302464</td>
<td align="center"><a class="uri" href="https://api.discogs.com/artists/302464">https://api.discogs.com/artists/302464</a></td>
<td align="center">Vinyl</td>
</tr>
</tbody>
</table>
<table style="width: 56%;"><colgroup> <col width="55%" /> </colgroup>
<thead>
<tr class="header">
<th align="center">resource_url</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center"><a class="uri" href="https://api.discogs.com/releases/5181773">https://api.discogs.com/releases/5181773</a></td>
</tr>
<tr class="even">
<td align="center"><a class="uri" href="https://api.discogs.com/releases/3690646">https://api.discogs.com/releases/3690646</a></td>
</tr>
<tr class="odd">
<td align="center"><a class="uri" href="https://api.discogs.com/releases/944917">https://api.discogs.com/releases/944917</a></td>
</tr>
<tr class="even">
<td align="center"><a class="uri" href="https://api.discogs.com/releases/906983">https://api.discogs.com/releases/906983</a></td>
</tr>
<tr class="odd">
<td align="center"><a class="uri" href="https://api.discogs.com/releases/8112758">https://api.discogs.com/releases/8112758</a></td>
</tr>
<tr class="even">
<td align="center"><a class="uri" href="https://api.discogs.com/releases/5800664">https://api.discogs.com/releases/5800664</a></td>
</tr>
</tbody>
</table>
<h3>I can’t see, I can’t see I’m going blind</h3>
And now, let's start visualising!
<h4>Most frequent labels</h4>
<pre class="r"><code>library(ggplot2)
ggplot(as.data.frame(head(sort(table(collection$label), decreasing = TRUE), 10)), aes(x = reorder(Var1, Freq), y = Freq)) + 
  geom_bar(stat = "identity", fill = "#B79477") + 
  coord_flip() + 
  xlab("Label") +
  ylab("Fréquence") +
  ggtitle("Labels les plus fréquents")</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2016/08/labels.jpeg"><img class="aligncenter size-full wp-image-1058" src="http://colinfay.me/wp-content/uploads/2016/08/labels.jpeg" alt="labels les plus représentés dans la collection discogs" width="800" height="400" /></a>

Philips and Polydor, <em>what a surprise!</em>
<h4>Most frequent artists</h4>
<pre class="r"><code>ggplot(as.data.frame(head(sort(table(collection$artist_name), decreasing = TRUE), 10)), aes(x = reorder(Var1, Freq), y = Freq)) + 
  geom_bar(stat = "identity", fill = "#B79477") + 
  coord_flip() + 
  xlab("Artistes") +
  ylab("Fréquence") +
  ggtitle("Artistes les plus fréquents")</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2016/08/artistes.jpeg"><img class="aligncenter size-full wp-image-1059" src="http://colinfay.me/wp-content/uploads/2016/08/artistes.jpeg" alt="Artistes les plus représentés" width="800" height="400" /></a>

So... here's the big revelation: I love Serge Gainsbourg and Geogres Brassens (guilty pleasure).
<h4>Release date</h4>
<pre class="r"><code>ggplot(dplyr::filter(collection, year != 0), aes(x = year)) + 
  geom_bar(stat = "count", fill = "#B79477") + 
  xlab("Année de sortie") +
  ylab("Fréquence") +
  ggtitle("Date de sorties des vinyles de la collection")</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2016/08/date-sortie.jpeg"><img class="aligncenter size-full wp-image-1060" src="http://colinfay.me/wp-content/uploads/2016/08/date-sortie.jpeg" alt="Dates de sorties" width="800" height="400" /></a>

Looks like I'm not a big 90's fan! My library show a bimodal distribution, with one mode around the 80's, and one around 2005.
<h3>It’s time to go deeper</h3>
So, can we get more information about this library?
<div id="major-tom-to-discogs-api" class="section level4">
<h4>Hello, it's me again</h4>
<pre class="r"><code>collection_2 &lt;- lapply(as.list(collection$release_id), function(obj){
  url &lt;- httr::GET(paste0("https://api.discogs.com/releases/", obj))
  url &lt;- rjson::fromJSON(rawToChar(url$content))
  data.frame(release_id = obj, 
             label = url$label[[1]]$name %||% NA,
             year = url$year %||% NA, 
             title = url$title %||% NA, 
             artist_name = url$artist[[1]]$name %||% NA, 
             styles = url$styles[[1]] %||% NA,
             genre = url$genre[[1]] %||% NA,
             average_note = url$community$rating$average %||% NA, 
             votes = url$community$rating$count %||% NA, 
             want = url$community$want %||% NA, 
             have = url$community$have %||% NA, 
             lowest_price = url$lowest_price %||% NA, 
             country = url$country %||% NA)
}) %&gt;% do.call(rbind, .) %&gt;% 
  unique()</code></pre>
Here, I have used the <em>release_id</em> element to make a request about each vinyl.

</div>
<h4>Most frequent genre</h4>
<pre class="r"><code>ggplot(as.data.frame(head(sort(table(collection_2$genre), decreasing = TRUE), 10)), aes(x = reorder(Var1, Freq), y = Freq)) + 
  geom_bar(stat = "identity", fill = "#B79477") + 
  coord_flip() + 
  xlab("Genre") +
  ylab("Fréquence") +
  ggtitle("Genres les plus fréquents")</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2016/08/genres-frequents.jpeg"><img class="aligncenter size-full wp-image-1062" src="http://colinfay.me/wp-content/uploads/2016/08/genres-frequents.jpeg" alt="Genres les plus fréquents" width="800" height="400" /></a>OH GOD, what a surprise! Almost half of my collection is made of rock albums (who could have guessed?).
<h4>Countries</h4>
<pre class="r"><code>ggplot(as.data.frame(head(sort(table(collection_2$country), decreasing = TRUE), 10)), aes(x = reorder(Var1, Freq), y = Freq)) + 
  geom_bar(stat = "identity", fill = "#B79477") + 
  coord_flip() + 
  xlab("Pays d'origine") +
  ylab("Fréquence") +
  ggtitle("Pays les plus fréquents")</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2016/08/pays-origine.jpeg"><img class="aligncenter size-large wp-image-1063" src="http://colinfay.me/wp-content/uploads/2016/08/pays-origine-1024x512.jpeg" alt="Pays d'origine des vinyles" width="809" height="405" /></a>
<h4>Average note</h4>
<pre class="r"><code>ggplot(collection_2, aes(x = average_note)) + 
  geom_histogram(fill = "#B79477") +
  xlab("Note moyenne") +
  ylab("Fréquence") +
  ggtitle("Notes moyennes des vinyles de la collection")</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2016/08/note-moyenne-vinyles-collection.jpeg"><img class="aligncenter wp-image-1073 size-full" src="http://colinfay.me/wp-content/uploads/2016/08/note-moyenne-vinyles-collection.jpeg" width="800" height="400" /></a>

Thanks a lot Discogs! It looks like I've got quite good musical tastes (thanks for the ego boost :) !)
<h4>Prices of vinyls (low range)</h4>
<pre class="r"><code>ggplot(collection_2, aes(x = lowest_price)) + 
  geom_histogram(fill = "#B79477") +
  xlab("Prix le plus bas") +
  ylab("Fréquence") +
  ggtitle("Prix le plus bas des vinyles de la collection")</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2016/08/densite-prix-bas.jpeg"><img class="aligncenter size-full wp-image-1072" src="http://colinfay.me/wp-content/uploads/2016/08/densite-prix-bas.jpeg" alt="densite-prix-bas" width="800" height="400" /></a>

Ok, I'm not gonna be rich selling my vinyl collection...
<h3>Let’s finish!</h3>
<pre class="r"><code>collection_complete &lt;- merge(collection, collection_2, by = c("release_id","label", "year", "title", "artist_name"))</code></pre>
<h4>Relationship between price and "want"</h4>
<pre class="r"><code>lm_want &lt;- lm(formula = lowest_price ~ want, data = collection_complete)
summary(lm_want)</code></pre>
<pre><code>##Residuals:
##   Min     1Q Median     3Q    Max 
##-8.043 -4.628 -2.224  2.179 49.608 

##Coefficients:
##            Estimate Std. Error t value Pr(&gt;|t|)    
##(Intercept) 6.715418   0.450582  14.904  &lt; 2e-16 ***
##want        0.005004   0.001788   2.799  0.00546 ** 
##---
##Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

##Residual standard error: 7.306 on 301 degrees of freedom
##  (5 observations deleted due to missingness)
##Multiple R-squared:  0.02536,    Adjusted R-squared:  0.02213 
##F-statistic: 7.833 on 1 and 301 DF,  p-value: 0.005461
</code></pre>
Here, we can see a correlation between the price and the number of users that put a "want" on a particular vinyl.

<code>ggplot(collection_complete, aes(x = lowest_price, y = want)) + geom_point(size = 3, color = "#B79477") + geom_smooth(method = "lm") + xlab("Prix le plus bas") + ylab("Nombre de \"want\"") + ggtitle("Prix et \"want\" des vinyles de la collection")</code>

<a href="http://colinfay.me/wp-content/uploads/2016/08/prix-wants-vinyls-collection.jpeg"><img class="aligncenter size-full wp-image-1076" src="http://colinfay.me/wp-content/uploads/2016/08/prix-wants-vinyls-collection.jpeg" alt="Prix en fonction du nombre de want" width="800" height="400" /></a>
<h4>Price and average note</h4>
<pre class="r"><code>lm_note &lt;- lm(formula = lowest_price ~ average_note, data = collection_complete)
lm_note$coefficients</code></pre>
<pre><code>##  (Intercept) average_note 
##    -1.504767     2.207834</code></pre>
Here, no significative correlation.
<pre class="r"><code>ggplot(collection_complete, aes(x = lowest_price, y = average_note)) + 
  geom_point(size = 3, color = "#B79477") +
  xlab("Prix le plus bas") +
  ylab("Note moyenne") +
  ylim(c(0,5)) +
  ggtitle("Prix et notes des vinyles de la collection")</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2016/08/prix-et-note-vinyles-collection.jpeg"><img class="aligncenter size-full wp-image-1080" src="http://colinfay.me/wp-content/uploads/2016/08/prix-et-note-vinyles-collection.jpeg" alt="Prix en fonction des notes" width="800" height="400" /></a>
<h3>And to conclude...</h3>
Next step... create a package to access the Discogs API? Why not! Let's put this on my to-do...
