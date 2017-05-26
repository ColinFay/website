---
ID: 1164
post_title: 'All I want for Christmas is a #Dataviz'
author: colin_fay
post_date: 2016-12-23 17:44:55
post_excerpt: ""
layout: single
permalink: /all-i-want-for-christmas-is-a-dataviz/
published: true
---
## Avant de vous lancer dans le marathon de repas de Noël, deux visualisations de données tirées de l'API lastfm... pour la requête "Christmas" !
<!--more-->
<div id="allo-lastfm" class="section level3">
### Allô LastFM
La première étape ? Créer un compte sur l’API LastFM, afin d’obtenir une clé d'accès. Une fois cette suite de caractères en poche, direction les requêtes avec R.

Commençons par charger les packages que nous allons utiliser :
```{r}library(tidyverse)
library(scales)
library(data.table)
library(magrittr)```
Les trois paramètres de départ sont :
```{r}#Le terme de recherche
query &lt;- "christmas"
#La clé API (cachée ici)
apikey &lt;- "XXX"
#L'index des pages de l'API, qui s'incrémentera au fur et à mesure des requêtes
x &lt;- 0```
Ensuite, nous entrons l’url de recherche. La recherche est limitée à 1000 résultats, et est divisible en pages avec l’argument "<code>&amp;page="</code>.
```{r}url &lt;- paste0("http://ws.audioscrobbler.com/2.0/?method=track.search&amp;track=", 
              query,"&amp;api_key=", apikey, "&amp;format=json","&amp;page=", x)```
Création d’une liste pour recevoir les résultats des requêtes, et requête sur la première page (index = 0).
```{r}dl &lt;- list()
dl2 &lt;- httr::GET(url)$content %&gt;%
  rawToChar() %&gt;% 
  rjson::fromJSON()
dl2 &lt;- dl2$results$trackmatches$track
dl &lt;- c(dl,dl2)```
Bouclons sur toutes les pages qui renvoient des résultats :
```{r}repeat{
  x &lt;- x+1
  url &lt;- paste0("http://ws.audioscrobbler.com/2.0/?method=track.search&amp;track=", 
                query,"&amp;api_key=", apikey, "&amp;format=json","&amp;limit=", 1000, "&amp;page=", x)
  dl2 &lt;- httr::GET(url)$content %&gt;%
    rawToChar() %&gt;% 
    rjson::fromJSON()
  dl2 &lt;- dl2$results$trackmatches$track
  dl &lt;- c(dl, dl2)
  if(length(dl2) == 0){
    break
  }
}```
Et enfin, créons le data.frame de résultats :
```{r}songs &lt;- lapply(dl, function(x){
  data.frame(name = x$name, 
             artist = x$artist, 
             listeners = as.numeric(x$listeners))
}) %&gt;%
  do.call(rbind, .) %&gt;% 
  arrange(listeners)```
</div>
<div id="and-now-lets-see" class="section level3">
### And now, let’s see!
Commençons par afficher les 5 titres écoutés par le plus d’utilisateurs :<code> </code>
```{r}songs &lt;- as.data.table(songs)
songs &lt;- songs[, .(listeners = mean(listeners)), by = .(name,artist)]
songs[1:15,] %&gt;%
ggplot(aes(x = reorder(name, listeners), y = listeners)) +
  geom_bar(stat = "identity", fill = "#d42426") +
  geom_text(data = songs[1:15,], aes(label= artist), size = 5, nudge_y = -sd(songs$listeners[1:15])/2) + 
  coord_flip() + 
  xlab("") +
  ylab("Volumes d'utilisateurs ayant écouté ce titre") +
  scale_y_continuous(labels = comma) +
  labs(title = "15 titres les plus populaires sur lastfm pour la requête Christmas", 
       subtitle = " ",
       caption = "http://colinfay.me") + 
  theme(axis.text=element_text(size=10),
        axis.title=element_text(size=15),
        title=element_text(size=18),
        plot.title=element_text(margin=margin(0,0,20,0), size=18),
        axis.title.x=element_text(margin=margin(20,0,0,0)),
        axis.title.y=element_text(margin=margin(0,20,0,0)),
        legend.text=element_text(size = 12),
        plot.margin=margin(20,20,20,20), 
        panel.background = element_rect(fill = "white"), 
        panel.grid.major = element_line(colour = "grey"))```
<a href="https://colinfay.github.io/wp-content/uploads/2016/12/songs-last-fm-christmas.jpeg"><img class="aligncenter size-large wp-image-1186" src="https://colinfay.github.io/wp-content/uploads/2016/12/songs-last-fm-christmas-1024x512.jpeg" alt="songs-last-fm-christmas" width="809" height="405" /></a>(cliquez pour zoomer)

Quant aux artistes les plus présents :
```{r}songs$artist %&gt;%
  table() %&gt;%
  as.data.frame() %&gt;%
  arrange(desc(Freq)) %&gt;%
  head(15) %&gt;%
ggplot(aes(x = reorder(., Freq), y = Freq)) +
  geom_bar(stat = "identity", fill = "#d42426") +
  coord_flip() + 
  xlab("") +
  ylab("Volume de titres de l'artiste") +
  scale_y_continuous(labels = comma) +
  labs(title = "15 artistes les plus présents sur lastfm pour la requête Christmas", 
       subtitle = " ",
       caption = "http://colinfay.me") + 
  theme(axis.text=element_text(size=10),
        axis.title=element_text(size=15),
        title=element_text(size=18),
        plot.title=element_text(margin=margin(0,0,20,0), size=18),
        axis.title.x=element_text(margin=margin(20,0,0,0)),
        axis.title.y=element_text(margin=margin(0,20,0,0)),
        legend.text=element_text(size = 12),
        plot.margin=margin(20,20,20,20), 
        panel.background = element_rect(fill = "white"), 
        panel.grid.major = element_line(colour = "grey"))```
<a href="https://colinfay.github.io/wp-content/uploads/2016/12/artist-christmas-lastfm.jpeg"><img class="aligncenter size-large wp-image-1184" title="" src="https://colinfay.github.io/wp-content/uploads/2016/12/artist-christmas-lastfm-1024x512.jpeg" alt="artists christmas last fm" width="809" height="405" /></a>(cliquez pour zoomer)

Et sur ce… joyeux Noël !

<a title="" href="https://colinfay.github.io/wp-content/uploads/2016/12/b546c88a28a7c2423d2a32bc85d1f106.gif"><img class="aligncenter size-full wp-image-1182" title="" src="https://colinfay.github.io/wp-content/uploads/2016/12/b546c88a28a7c2423d2a32bc85d1f106.gif" alt="Nightmare before christmas" width="500" height="301" /></a>

</div>
