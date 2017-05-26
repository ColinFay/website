---
ID: 1697
post_title: '#RStats & API Microsoft Computer Vision API'
author: colin_fay
post_date: 2017-05-10 14:00:58
post_excerpt: ""
permalink: /rstats-microsoft-computer-vision-api-2/
published: true

---

<h2>Exploration des photos de profils des twittos #RStats avec l'API Microsoft Computer Vision.</h2>
<!--more-->

<span class="notranslate">Ce billet de blog s'inspire du travail de Maelle Salmon avec <a href="http://www.masalmon.eu/2017/03/19/facesofr/" target="_blank" rel="noopener noreferrer">Faces of #RStats twitter</a> et d'un article sur Data Bzh utilisant l'API Microsoft Computer Vision pour examiner d'anciennes <a href="http://data-bzh.fr/photographies-fonds-de-la-guerre-14-18-en-bretagne/" target="_blank" rel="noopener noreferrer">photos de Bretagne</a> .</span>
<h3>Microsoft Computer Vision</h3>
<span class="notranslate">Cette <a href="https://www.microsoft.com/cognitive-services/en-us/computer-vision-api" target="_blank" rel="noopener noreferrer">API</a> est utilisée pour décrire et étiqueter automatiquement une image.</span> <span class="notranslate">Voici comment vous pouvez l'utiliser avec R, avec un jeu de données regroupant des images de profils Twitter.</span>
<h3>Les visages #RStats — Étiquetage automatique</h3>
<span class="notranslate">Dans cet article, vous trouverez un tuto sur comment obtenir des photos de profil de Twitter et les étiqueter automatiquement avec Microsoft Computer Vision.</span>
<h4>Collecter les données</h4>
<pre class="r"><code>library(tidyverse)
library(rtweet)
library(httr)
library(jsonlite)
token &lt;- create_token( app = "XX", consumer_key = "XXX", consumer_secret = "XX</code><span style="font-family: 'Noto Serif', sans-serif;">")</span></pre>
<pre class="r"><code>users &lt;- search_users(q= '#rstats',
                      n = 1000,
                      parse = TRUE) %&gt;%
  unique()</code></pre>
<p style="text-align: right;"><em>Note: J'ai ici anonymisé mes API keys.</em></p>
Maintenant, utilisons la colonne <code>profile_image_url</code> pour obtenir l'url des photos de profil.

<span class="notranslate">D'abord, cette variable a besoin d'être nettoyée : les URL contiennent un paramètre <em>_normal</em>, créant des images 48x48.</span> <span class="notranslate">L'API Microsoft a besoin d'une résolution minimum de 50x50, nous devons donc nous débarrasser de ce paramètre.</span>
<pre class="r"><code>users$profile_image_url &lt;- gsub("_normal", "", users$profile_image_url)</code></pre>
<h4>Interroger l'API  Microsoft</h4>
D'abord, inscrivez-vous sur <a href="https://www.microsoft.com/cognitive-services/en-us/computer-vision-api" target="_blank" rel="noopener noreferrer">Microsoft API service</a>, <span class="notranslate">et lancez un essai gratuit.</span> <span class="notranslate">Ce compte gratuit est limité: vous ne pouvez faire que 5 000 appels par mois et 20 par minute.</span> <span class="notranslate">Mais c'est bien assez pour notre cas (478 images à regarder).</span>
<pre class="r"><code>users_api &lt;- lapply(users[,25],function(i, key = "") {
  request_body &lt;- data.frame(url = i)
  request_body_json &lt;- gsub("\\[|\\]", "", toJSON(request_body, auto_unbox = "TRUE"))
  result &lt;- POST("https://api.projectoxford.ai/vision/v1.0/analyze?visualFeatures=Tags,Description,Faces,Categories",
                 body = request_body_json,
                 add_headers(.headers = c("Content-Type"="application/json","Ocp-Apim-Subscription-Key"="XXX")))
  Output &lt;- content(result)
  if(length(Output$description$tags) != 0){
    cap &lt;- Output$description$captions
  } else {
    cap &lt;- NA
  }
  if(length(Output$description$tags) !=0){
    tag &lt;-list(Output$description$tags)
  }
  d &lt;- tibble(cap, tag)
  Sys.sleep(time = 3)
  return(d)
})%&gt;%
  do.call(rbind,.)</code></pre>
<p style="text-align: right;"><em><span class="notranslate">Remarque: J'ai (à nouveau) caché ma clé API.</span>
<span class="notranslate">Ce code peut prendre un certain temps à exécuter, car il contient un appel à la fonction Sys.sleep.</span> <span class="notranslate">Pour en savoir plus</span>, <a href="http://colinfay.me/rstats-api-calls-sys-sleep/" target="_blank" rel="noopener noreferrer">lire ce billet</a>. </em></p>

<h4>Créer des tibbles</h4>
<span class="notranslate">Maintenant, j'ai un tibble avec une colonne contenant les listes de légendes et de score de confiance, et une colonne avec les listes des balises associées à chaque image</span><span class="notranslate">.</span>
<pre class="r"><code>users_cap &lt;- lapply(users_api$cap, unlist) %&gt;%
  do.call(rbind,.) %&gt;%
  as.data.frame() 
users_cap$confidence &lt;- as.character(users_cap$confidence) %&gt;%
  as.numeric()
users_tags &lt;- unlist(users_api$tag) %&gt;%
  data.frame(tag = .)</code></pre>
<h3>Visualisation</h3>
<span class="notranslate">Chaque légende est donnée avec un score de confiance.</span>
<pre class="r"><code>ggplot(users_cap, aes(as.numeric(confidence))) +
  geom_histogram(fill = "#b78d6a", bins = 50) + 
  xlab("Confidence") + 
  ylab("") + 
  labs(title = "Faces of #RStats - Captions confidence", 
       caption="http://colinfay.me") + 
  theme_light()</code></pre>
[caption id="attachment_1583" align="aligncenter" width="1000"]<a href="https://colinfay.github.io/wp-content/uploads/2017/04/rstats-caption-confidence.png"><img class="size-full wp-image-1583" src="https://colinfay.github.io/wp-content/uploads/2017/04/rstats-caption-confidence.png" alt="" width="1000" height="500" /></a> Cliquez pour zoomer[/caption]

<span class="notranslate">Il semble que les scores de confiance pour les légendes ne soient pas très forts. </span>

<span class="notranslate">R</span><span class="notranslate">egardons les légendes et les balises les plus fréquentes.</span>
<pre class="r"><code>users %&gt;%
  group_by(text)%&gt;%
  summarize(somme = sum(n())) %&gt;%
  arrange(desc(somme))%&gt;%
  na.omit() %&gt;%
  .[1:25,] %&gt;%
  ggplot(aes(reorder(text, somme), somme)) +
  geom_bar(stat = "identity",fill = "#b78d6a") +
  coord_flip() +
  xlab("") + 
  ylab("") + 
  labs(title = "Faces of #RStats - Captions", 
       caption="http://colinfay.me") +   
  theme_light()</code></pre>
[caption id="attachment_1580" align="aligncenter" width="800"]<a href="https://colinfay.github.io/wp-content/uploads/2017/04/rstats-captions-users.png"><img class="size-full wp-image-1580" src="https://colinfay.github.io/wp-content/uploads/2017/04/rstats-captions-users.png" alt="" width="800" height="400" /></a> Click to zoom[/caption]

<span class="notranslate">Eh bien ... Je ne suis pas sûr qu'il y ait tant de passionnés de surf et de skate dans notre liste, mais soit...</span>
<pre class="r"><code>users_tags %&gt;%
  group_by(tag)%&gt;%
  summarize(somme = sum(n())) %&gt;%
  arrange(desc(somme))%&gt;%
  .[1:25,] %&gt;%
  ggplot(aes(reorder(tag, somme), somme)) +
  geom_bar(stat = "identity",fill = "#b78d6a") +
  coord_flip() +
  xlab("") + 
  ylab("") + 
  labs(title = "Faces of #RStats - Tags", 
       caption="http://colinfay.me") +   
  theme_light()

</code></pre>
<h2><a href="https://colinfay.github.io/wp-content/uploads/2017/04/rstats-tags.png"><img class="aligncenter size-full wp-image-1584" src="https://colinfay.github.io/wp-content/uploads/2017/04/rstats-tags.png" alt="" width="1000" height="500" /></a></h2>
<h2>Quelques vérifications</h2>
Jetons un coup d’œil à l'image avec le score de confiance le plus élevé, avec la légende que l'API lui a donnée.

[caption id="attachment_1459" align="aligncenter" width="300"]<a href="https://colinfay.github.io/wp-content/uploads/2017/04/9mJTF0PO.jpeg"><img class="size-full wp-image-1459" src="https://colinfay.github.io/wp-content/uploads/2017/04/9mJTF0PO.jpeg" alt="" width="300" height="300" /></a> A man wearing a suit and tie — 0.92 confidence.[/caption]

<span class="notranslate">Il n'a pas de cravate, mais l'API a bien saisi le reste.</span>

<span class="notranslate">Et maintenant, juste pour le plaisir, la légende avec le score de confiance le plus bas :</span>

[caption id="attachment_1460" align="aligncenter" width="300"]<a href="https://colinfay.github.io/wp-content/uploads/2017/04/czR2-o0M.jpg"><img class="size-full wp-image-1460" src="https://colinfay.github.io/wp-content/uploads/2017/04/czR2-o0M.jpg" alt="" width="300" height="300" /></a> A close up of two giraffes near a tree - 0.02 confidence[/caption]

Bien vu ;)

<span class="notranslate">Pour une vérification plus plus systémique, regardons un collage d'images, réalisé à partir des légendes les plus fréquentes.</span>
<p style="text-align: right;"><em>Remarque: afin de se concentrer sur les détails des images et de se débarrasser du genre des légendes, j'ai remplacé "man / woman / men / womens" par "persoe / persons" dans l'ensemble de données, avant de créer ces collages. </em></p>


[caption id="attachment_1533" align="aligncenter" width="840"]<a href="https://colinfay.github.io/wp-content/uploads/2017/04/caption_man_skatepark.jpg"><img class="size-large wp-image-1533" src="https://colinfay.github.io/wp-content/uploads/2017/04/caption_man_skatepark-1024x1024.jpg" alt="" width="840" height="840" /></a> A person on a surf board in a skate park[/caption]

&nbsp;

[caption id="attachment_1556" align="aligncenter" width="840"]<a href="https://colinfay.github.io/wp-content/uploads/2017/04/smiling_camera.jpg"><img class="size-large wp-image-1556" src="https://colinfay.github.io/wp-content/uploads/2017/04/smiling_camera-1024x514.jpg" alt="" width="840" height="422" /></a> A person is smiling at the camera - Confidence mean : 0.54[/caption]

&nbsp;

[caption id="attachment_1535" align="aligncenter" width="840"]<a href="https://colinfay.github.io/wp-content/uploads/2017/04/caption_girafe.jpg"><img class="size-large wp-image-1535" src="https://colinfay.github.io/wp-content/uploads/2017/04/caption_girafe-1024x514.jpg" alt="" width="840" height="422" /></a> A close up of two giraffes near a tree — Confidence mean : 0.0037[/caption]

&nbsp;

[caption id="attachment_1557" align="aligncenter" width="840"]<a href="https://colinfay.github.io/wp-content/uploads/2017/04/mosaic_glasses.jpg"><img class="size-large wp-image-1557" src="https://colinfay.github.io/wp-content/uploads/2017/04/mosaic_glasses-1024x514.jpg" alt="" width="840" height="422" /></a> A person wearing glasses looking at the camera[/caption]

<span class="notranslate">Les premier et troisième collages sont clairement erronés sur les légendes.</span> Mais<span class="notranslate">, nous pouvons voir que le score de confiance y est très bas.</span> <span class="notranslate">Le deuxième et le quatrième, cependant, semblent être plus précis.</span> <span class="notranslate">Peut-être que nous devons essayer à nouveau avec d'autres images, juste pour être sûr ... Mais ça sera pour une autre fois 😉</span>
