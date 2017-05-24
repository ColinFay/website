---
ID: 1249
post_title: 'All I want for Christmas is a #Dataviz'
author: colin_fay
post_date: 2016-12-23 17:44:55
post_excerpt: ""
layout: post
permalink: >
  /all-i-want-for-christmas-is-a-dataviz-2/
published: true
---
<h2>Just before Christmas, let'senjoy these two visualisations created with data from the lastfm API.</h2>
<!--more-->
<h3>Allô LastFM</h3>
First, you need to create an account on <a href="http://www.last.fm/api" target="_blank">lastfm</a>, and get an access key. When you have it, you can start making resquests on the API with R.

Let's load the packages we will need:
<pre class="r"><code>library(tidyverse)
library(scales)
library(data.table)
library(magrittr)</code></pre>
Here are the three parameters you will need before starting:
<pre class="r"><code>#The query
query &lt;- "christmas"
#Your API key (masked here)
apikey &lt;- "XXX"
#The page index
x &lt;- 0</code></pre>
Then, the search url. Each request is limited to 1000 answers. The results are divided in pages, and you can access them with the arg "<code>&amp;page="</code>.
<pre class="r"><code>url &lt;- paste0("http://ws.audioscrobbler.com/2.0/?method=track.search&amp;track=", 
              query,"&amp;api_key=", apikey, "&amp;format=json","&amp;page=", x)</code></pre>
Let's create a list to concatenate our results, and query the first page (index = 0).
<pre class="r"><code>dl &lt;- list()
dl2 &lt;- httr::GET(url)$content %&gt;%
  rawToChar() %&gt;% 
  rjson::fromJSON()
dl2 &lt;- dl2$results$trackmatches$track
dl &lt;- c(dl,dl2)</code></pre>
Then, we loop over all the pages:
<pre class="r"><code>repeat{
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
}</code></pre>
And now, let's create the dataframe:
<pre class="r"><code>songs &lt;- lapply(dl, function(x){
  data.frame(name = x$name, 
             artist = x$artist, 
             listeners = as.numeric(x$listeners))
}) %&gt;%
  do.call(rbind, .) %&gt;% 
  arrange(listeners)</code></pre>
<h3>And now, let’s see!</h3>
The fifteen most popular songs are:<code> </code>
<pre class="r"><code>songs &lt;- as.data.table(songs)
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
        panel.grid.major = element_line(colour = "grey"))</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2016/12/songs-last-fm-christmas.jpeg"><img class="aligncenter size-large wp-image-1186" src="http://colinfay.me/wp-content/uploads/2016/12/songs-last-fm-christmas-1024x512.jpeg" alt="songs-last-fm-christmas" width="809" height="405" /></a>(click to zoom)

And the most frequent artists:
<pre class="r"><code>songs$artist %&gt;%
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
        panel.grid.major = element_line(colour = "grey"))</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2016/12/artist-christmas-lastfm.jpeg"><img class="aligncenter size-large wp-image-1184" title="" src="http://colinfay.me/wp-content/uploads/2016/12/artist-christmas-lastfm-1024x512.jpeg" alt="artists christmas last fm" width="809" height="405" /></a>(click to zoom)

So now... Merry Christmas!

<a title="" href="http://colinfay.me/wp-content/uploads/2016/12/b546c88a28a7c2423d2a32bc85d1f106.gif"><img class="aligncenter size-full wp-image-1182" title="" src="http://colinfay.me/wp-content/uploads/2016/12/b546c88a28a7c2423d2a32bc85d1f106.gif" alt="Nightmare before christmas" width="500" height="301" /></a>