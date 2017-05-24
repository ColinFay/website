---
ID: 1603
post_title: '#RStats — API calls et Sys.sleep'
author: colin_fay
post_date: 2017-04-14 13:00:53
post_excerpt: ""
layout: single
permalink: >
  /rstats-api-calls-sys-sleep/
published: true
---
<h2>J'ai récemment reçu un mail concernant mon post sur l'API Discogs, disant que le code ne fonctionnait pas.</h2>
<h2>Il s'est avéré que cela était dû aux nouvelles limitations de l'API. Voici comment contourner ces limites avec R.
<!--more--></h2>
<h3>Requête Discogs</h3>
Voici <a href="http://colinfay.me/data-vinyles-bibliotheque-discogs-r/" target="_blank">le billet de blog décrivant comment faire des requêtes sur l'API Discogs avec R</a>.

Il y a quelques semaines, j'ai reçu un mail indiquant que la deuxième partie du code ne fonctionnait pas, renvoyant seulement des <em>NA</em>. Il s'est avéré que cela était dû aux changements récents de l'API Discogs, limitant le nombre de requêtes possibles dans une période spécifique.

Récemment également, j'ai utilisé l'API <em>Computer Vision</em> de Microsoft - une API limitant les requêtes à 20 par minute -  pour un blogpost sur <a href="http://data-bzh.fr">Data Bzh</a>.

Alors, comment automatiser le requêtage sur une API quand elle limite le nombre de requêtes par minutes / heure / jour (par exemple, si vous avez 282 images à analyser) ?
<h3>Première méthode : for loop et Sys.sleep()</h3>
<em>Remarque : ce blogpost ne contient aucune requête API, j'utiliserais Sys.time () pour montrer comment fonctionne Sys.sleep ().
</em>

Si vous souhaitez limiter à 20 appels par minute, vous devrez utiliser <em>Sys.sleep ()</em>. Cette fonction ne prend qu'un argument, <em>time</em>, qui est le nombre de secondes que vous souhaitez arrêter R avant de reprendre.

Par exemple, dans un for loop, vous pouvez faire une pause de 10 secondes à chaque itération de la boucle :
<pre class="r"><code>for(i in 1:3){
  print(Sys.time())
  Sys.sleep(time = 10)
}</code></pre>
<pre><code>## [1] "2017-03-26 11:13:58 CET"
## [1] "2017-03-26 11:14:08 CET"
## [1] "2017-03-26 11:14:18 CET"</code></pre>
<h3>Avec un lapply</h3>
Si vous avez accès au cœur de la fonction que vous souhaitez utiliser (par exemple la fonction requêtant l'API), vous pouvez utiliser un <em>lapply</em>, et insérer <em>Sys.sleep ()</em> dans cette fonction.

C'est cette méthode que vous devrez utiliser dans le code pour Discogs, et que j'ai utiliser dans le billet sur Computer Vision.
<pre class="r"><code>library(tidyverse)</code>
<code>lapply(1:3, function(x) {
  print(x)
  print(Sys.time()) 
  Sys.sleep(3)
}) %&gt;% do.call(rbind, .) </code></pre>
<pre><code>## [1] 1
## [1] "2017-03-26 11:20:22 CET"
## [1] 2
## [1] "2017-03-26 11:20:25 CET"
## [1] 3
## [1] "2017-03-26 11:20:28 CET"
</code></pre>
<em>Hope this can help!</em>
