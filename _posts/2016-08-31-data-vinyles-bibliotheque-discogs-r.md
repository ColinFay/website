---
ID: 1040
post_title: 'Data &#038; Vinyles — Exploration d&rsquo;une bibliothèque Discogs avec R'
author: colin_fay
post_date: 2016-08-31 12:00:36
post_excerpt: ""
layout: single
permalink: /data-vinyles-bibliotheque-discogs-r/
published: true
---
<h2>Amoureux de données et de vinyles, je me suis amusé à envoyer quelques requêtes sur l’API Discogs avec R, pour en savoir un peu plus sur ma collection.</h2>
<!--more-->

Réseau social incontournable des amateurs du disque microsillon, Discogs offre une API permettant de jongler entre musique et données en quelques lignes de code.

<span style="text-decoration: underline;">Note</span> : pour retrouver les données utilisées dans ce billet, vous pouvez télécharger <a href="http://colinfay.me/data/collection_complete.json" target="_blank">le document JSON ici</a>, ou directement dans R :
<pre class="r"><code>collection_complete &lt;- jsonlite::fromJSON(txt = "http://colinfay.me/data/collection_complete.json", simplifyDataFrame = TRUE)</code></pre>
<h3>Major Tom to Discogs API</h3>
Avant de se lancer, chargeons dans l’environnement deux fonctions qui me seront indispensables : <em><code>%&gt;% </code></em>et <em><code>%||%</code>.</em>
<pre class="r"><code>library(magrittr)
`%||%` &lt;- function(a,b) if(is.null(a)) b else a</code></pre>
Bien bien, commençons par faire venir le profil Discogs :
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
Cette première requête nous permet d’obtenir les informations basiques sur l’utilisateur (en l’occurrence “_colin“, principal intéressé de ce billet de blog).

<code>$count</code> nous apprend notamment que la collection compte 308 entrées. L’élément de la liste <code>$id</code> renvoie le nombre de “folders” Discogs créées par l’utilisateur – ici, 0 correspond à l’ensemble de la collection, sans spécification de liste.
<h3>Créer un data.frame avec l’ensemble des vinyles présents dans la collection</h3>
L’API Discogs permet d'accéder à des pages de 100 résultats maximum. Ma collection contenant 308 entrées, nous devons réaliser une boucle <code>repeat</code> qui collectera l’ensemble des données, avant de créer un tableau final contenant l’ensemble de la collection.
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
Et pour un aperçu du tableau obtenu :
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
<h3><em>I can’t see, I can’t see I’m going blind</em></h3>
Bon, il est temps de mettre tout cela en forme, non ?
<h4>Les labels les plus représentés</h4>
<pre class="r"><code>library(ggplot2)
ggplot(as.data.frame(head(sort(table(collection$label), decreasing = TRUE), 10)), aes(x = reorder(Var1, Freq), y = Freq)) + 
  geom_bar(stat = "identity", fill = "#B79477") + 
  coord_flip() + 
  xlab("Label") +
  ylab("Fréquence") +
  ggtitle("Labels les plus fréquents")</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2016/08/labels.jpeg"><img class="aligncenter size-full wp-image-1058" src="http://colinfay.me/wp-content/uploads/2016/08/labels.jpeg" alt="labels les plus représentés dans la collection discogs" width="800" height="400" /></a>

Philips et Polydor, <em>what a surprise!</em>
<h4>Les artistes les plus représentés</h4>
<pre class="r"><code>ggplot(as.data.frame(head(sort(table(collection$artist_name), decreasing = TRUE), 10)), aes(x = reorder(Var1, Freq), y = Freq)) + 
  geom_bar(stat = "identity", fill = "#B79477") + 
  coord_flip() + 
  xlab("Artistes") +
  ylab("Fréquence") +
  ggtitle("Artistes les plus fréquents")</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2016/08/artistes.jpeg"><img class="aligncenter size-full wp-image-1059" src="http://colinfay.me/wp-content/uploads/2016/08/artistes.jpeg" alt="Artistes les plus représentés" width="800" height="400" /></a>

Bon, voilà, vous le savez... j'aime beaucoup Serge Gainsbourg et Georges Brassens (j'assume). La présence forte de Various était prévisible : il s'agit d'un terme générique faisant référence aux compilations, il est donc plus facile de gonfler ce chiffre que celui d'un artiste "solo".
<h4>Date de sortie des vinyles de la collection</h4>
<pre class="r"><code>ggplot(dplyr::filter(collection, year != 0), aes(x = year)) + 
  geom_bar(stat = "count", fill = "#B79477") + 
  xlab("Année de sortie") +
  ylab("Fréquence") +
  ggtitle("Date de sorties des vinyles de la collection")</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2016/08/date-sortie.jpeg"><img class="aligncenter size-full wp-image-1060" src="http://colinfay.me/wp-content/uploads/2016/08/date-sortie.jpeg" alt="Dates de sorties" width="800" height="400" /></a>

Woaw, j'ai comme l'impression que je ne suis pas un super fan des années 90... Pourtant, il y a de <a href="https://www.youtube.com/watch?v=NcKAdFENqig" target="_blank">super titres</a>, non ? Ma collection se concentre sur un gros pic autour des années 80 et 00, avec un mode en 1980.
<h3>It’s time to go deeper</h3>
Bien, ces infos un temps soit peu basiques nous offrent déjà une première vision des vinyles dans ma collection. Et si l’on allait plus loin ?
<div id="major-tom-to-discogs-api" class="section level4">
<h4>Hello, it's me again</h4>
&nbsp;
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
Ici, nous utilisons le release_id pour aller à la pêche aux informations pour toutes les entrées de la base.

</div>
<h4>Les genres les plus représentés</h4>
<pre class="r"><code>ggplot(as.data.frame(head(sort(table(collection_2$genre), decreasing = TRUE), 10)), aes(x = reorder(Var1, Freq), y = Freq)) + 
  geom_bar(stat = "identity", fill = "#B79477") + 
  coord_flip() + 
  xlab("Genre") +
  ylab("Fréquence") +
  ggtitle("Genres les plus fréquents")</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2016/08/genres-frequents.jpeg"><img class="aligncenter size-full wp-image-1062" src="http://colinfay.me/wp-content/uploads/2016/08/genres-frequents.jpeg" alt="Genres les plus fréquents" width="800" height="400" /></a>OH GOD, quelle surprise ! Eh oui, presque la moitié de ma collection contient des vinyles de rock (tous styles confondus). Qui aurait pu s'en douter ?
<h4>Pays d’origine des vinyles</h4>
<pre class="r"><code>ggplot(as.data.frame(head(sort(table(collection_2$country), decreasing = TRUE), 10)), aes(x = reorder(Var1, Freq), y = Freq)) + 
  geom_bar(stat = "identity", fill = "#B79477") + 
  coord_flip() + 
  xlab("Pays d'origine") +
  ylab("Fréquence") +
  ggtitle("Pays les plus fréquents")</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2016/08/pays-origine.jpeg"><img class="aligncenter size-large wp-image-1063" src="http://colinfay.me/wp-content/uploads/2016/08/pays-origine-1024x512.jpeg" alt="Pays d'origine des vinyles" width="809" height="405" /></a>Bon, une large partie des vinyles venus de France, ça se tient :)
<h4>Notes moyennes des vinyles</h4>
<pre class="r"><code>ggplot(collection_2, aes(x = average_note)) + 
  geom_histogram(fill = "#B79477") +
  xlab("Note moyenne") +
  ylab("Fréquence") +
  ggtitle("Notes moyennes des vinyles de la collection")</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2016/08/note-moyenne-vinyles-collection.jpeg"><img class="aligncenter wp-image-1073 size-full" src="http://colinfay.me/wp-content/uploads/2016/08/note-moyenne-vinyles-collection.jpeg" width="800" height="400" /></a>

D'après la communauté Discogs j'ai plutôt de bons goûts musicaux... Merci pour cet <em>ego boost</em> ! (Oui, je compte ignorer la présence de 10 vinyles notés 0... Après tout, on a tous les droits à ses petits guilty pleasure...)
<h4>Money Money Money</h4>
<pre class="r"><code>ggplot(collection_2, aes(x = lowest_price)) + 
  geom_histogram(fill = "#B79477") +
  xlab("Prix le plus bas") +
  ylab("Fréquence") +
  ggtitle("Prix le plus bas des vinyles de la collection")</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2016/08/densite-prix-bas.jpeg"><img class="aligncenter size-full wp-image-1072" src="http://colinfay.me/wp-content/uploads/2016/08/densite-prix-bas.jpeg" alt="densite-prix-bas" width="800" height="400" /></a>

Bon... Ce n'est visiblement pas en vendant ma collection que je vais faire fortune. Cela dit, je ne comptais pas m'en séparer, cela tombe plutôt bien.
<h3>Let’s finish!</h3>
<pre class="r"><code>collection_complete &lt;- merge(collection, collection_2, by = c("release_id","label", "year", "title", "artist_name"))</code></pre>
<h4>Prix des vinyles en fonction des “want”</h4>
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
Ici, nous voyons que le coefficient de <em>want</em> est significatif, avec une probabilité critique de 0.005. Nous pouvons donc conclure que, pour cette discothèque, le volume de want est significativement lié à la variable prix.
<pre class="r"><code>ggplot(collection_complete, aes(x = lowest_price, y = want)) + 
  geom_point(size = 3, color = "#B79477") +
  geom_smooth(method = "lm") + 
  xlab("Prix le plus bas") +
  ylab("Nombre de \"want\"") +
  ggtitle("Prix et \"want\" des vinyles de la collection")</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2016/08/prix-wants-vinyls-collection.jpeg"><img class="aligncenter size-full wp-image-1076" src="http://colinfay.me/wp-content/uploads/2016/08/prix-wants-vinyls-collection.jpeg" alt="Prix en fonction du nombre de want" width="800" height="400" /></a>

En clair : sur Discogs, il est possible d'entrer des vinyles dans une "liste d'envie", labellisée "want" dans notre population. Ici, nous avons dessiné la régression linéaire du prix le plus bas en fonction du nombre de personnes ayant listé cette sortie dans leur liste d'envie. La tendance est légère, avec beaucoup de bruit lorsque nous nous rapprochons des prix les plus élevés.
<h4>Prix des vinyles en fonction des notes</h4>
<pre class="r"><code>lm_note &lt;- lm(formula = lowest_price ~ average_note, data = collection_complete)
lm_note$coefficients</code></pre>
<pre><code>##  (Intercept) average_note 
##    -1.504767     2.207834</code></pre>
Ici, le coefficient étant de 2.2, nous ne pouvons pas lier la variable note à la variable prix. Visualisons les données pour afficher la dispersion.
<pre class="r"><code>ggplot(collection_complete, aes(x = lowest_price, y = average_note)) + 
  geom_point(size = 3, color = "#B79477") +
  xlab("Prix le plus bas") +
  ylab("Note moyenne") +
  ylim(c(0,5)) +
  ggtitle("Prix et notes des vinyles de la collection")</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2016/08/prix-et-note-vinyles-collection.jpeg"><img class="aligncenter size-full wp-image-1080" src="http://colinfay.me/wp-content/uploads/2016/08/prix-et-note-vinyles-collection.jpeg" alt="Prix en fonction des notes" width="800" height="400" /></a>
<h3>And to conclude...</h3>
La prochaine étape ? Faire un package pour accéder à l'API... et pourquoi pas ? Je mets ça sur ma to-do !
