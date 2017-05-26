---
ID: 1418
post_title: 1 dataset, 10 visualisations with R
author: colin_fay
post_date: 2017-03-15 21:45:45
post_excerpt: ""
layout: single
permalink: /1-dataset-10-dataviz-with-r/
published: true
---
<h2>Creating 10 datavisualisations from 1 dataset with R.
<!--more--></h2>
<h3><em>Back from La fabrique de l’info</em></h3>
<a href="http://colinfay.me/wp-content/uploads/2017/03/IMG_2771.jpg"><img class="size-medium wp-image-1412 alignleft" src="http://colinfay.me/wp-content/uploads/2017/03/IMG_2771-300x225.jpg" alt="" width="300" height="225" /></a>On the 12th of march, I gave a speech in Rennes about data &amp; visualisation.

During this talk, I presented the audience with a series of dataviz made from one dataset, in order to show how you can get various information and points of view from the very same dataset, depending on what you want to show the reader.
<p style="text-align: center;"><a href="https://github.com/ColinFay/conf/blob/master/fabrique-info/La%20fabrique%20de%20l'info.pdf">Slides from my talk</a></p>
For this, I took this <a href="https://data.enseignementsup-recherche.gouv.fr/explore/dataset/fr-esr-pes-pedr-beneficiaires/">dataset</a>, and created 10 visualizations, each focusing on specific variable(s). Here's how to make them with R.
<h3>1 dataset, 10 dataviz</h3>
<em>Notes
</em>These plots have been created using tools from the <code>databzhtools</code> you can find here : <a class="uri" href="https://github.com/DataBzh/data-bzh-tools">https://github.com/DataBzh/Data-bzh-tools</a>
Download it on your hard drive, and load it in R with : <code>source("data-bzh-tools-master/main.R")</code>
<pre class="r"><code>#Charger les données
library(tidyverse)
source(file = "data-bzh-tools-master/main.R")
prim &lt;- read_csv2("https://data.enseignementsup-recherche.gouv.fr/explore/dataset/fr-esr-pes-pedr-beneficiaires/download/?format=csv&amp;timezone=Europe/Berlin&amp;use_labels_for_header=true")
prim$Année &lt;- paste0("01-01-",prim$Année) %&gt;%
  lubridate::dmy()</code></pre>
<h4>Frequency distribution</h4>
<pre class="r"><code>prim %&gt;%
  group_by(Année) %&gt;%
  summarise(somme = sum(Bénéficiaires))%&gt;%
  ggplot(aes(Année, somme)) + 
  geom_bar(stat = "identity", fill = databzh$colour1) + 
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()</code></pre>
<h3><a href="http://colinfay.me/wp-content/uploads/2017/03/Rplot06.jpeg"><img class="aligncenter size-large wp-image-1393" src="http://colinfay.me/wp-content/uploads/2017/03/Rplot06-1024x512.jpeg" alt="" width="840" height="420" /></a></h3>
<h4>Prop by sector</h4>
<pre class="r"><code>prim %&gt;%
  group_by(Année, `Secteur disciplinaire`) %&gt;%
  summarise(somme = sum(Bénéficiaires))%&gt;%
  ggplot(aes(Année, somme, fill = `Secteur disciplinaire`)) + 
  scale_fill_manual(values = databzh$colours) +
  geom_bar(stat = "identity", position = "fill") + 
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()</code></pre>
<h3><a href="http://colinfay.me/wp-content/uploads/2017/03/Rplot07.jpeg"><img class="aligncenter size-large wp-image-1394" src="http://colinfay.me/wp-content/uploads/2017/03/Rplot07-1024x512.jpeg" alt="" width="840" height="420" /></a></h3>
<h4>Value by sector</h4>
<pre class="r"><code>prim %&gt;%
  group_by(Année, Bénéficiaires, `Secteur disciplinaire`) %&gt;%
  summarise(somme = sum(Bénéficiaires))%&gt;%
  ggplot(aes(Année,somme, group = `Secteur disciplinaire`, col = `Secteur disciplinaire`)) + 
  geom_point() +
  scale_color_manual(values = databzh$colours) +
  facet_grid(~`Secteur disciplinaire`) +
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()</code></pre>
<h3><a href="http://colinfay.me/wp-content/uploads/2017/03/Rplot12.jpeg"><img class="aligncenter size-large wp-image-1395" src="http://colinfay.me/wp-content/uploads/2017/03/Rplot12-1024x512.jpeg" alt="" width="840" height="420" /></a></h3>
<h4>Map</h4>
<pre class="r"><code>#states &lt;- map_data("france")
prim2 &lt;- prim %&gt;%
  separate(`Géo-localisation`, into = c("lon","lat"), sep =',') %&gt;%
  group_by(lon, lat) %&gt;%
  summarise(somme = sum(Bénéficiaires))
ggplot(states, aes(long,lat, group=group)) + 
  geom_polygon(fill = "#e4e4e4") + 
  coord_map() +
  geom_path(data=states, aes(long, lat, group=group, fill=NULL), color="grey50") +
  geom_point(data = prim2, aes(x = as.numeric(lat), y = as.numeric(lon), group = NULL,size = somme, col = databzh$colour3)) + 
  scale_color_manual(values = palette) + 
  scale_size(range = c(1,10)) +
  xlim(range(range(states$long))) +
  ylim(range(range(states$lat))) +
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()</code></pre>
<h3><a href="http://colinfay.me/wp-content/uploads/2017/03/Rplot20.jpeg"><img class="aligncenter size-large wp-image-1396" src="http://colinfay.me/wp-content/uploads/2017/03/Rplot20-1024x512.jpeg" alt="" width="840" height="420" /></a></h3>
<h4>Boxplot</h4>
<pre class="r"><code>ggplot(prim, aes(Région, Bénéficiaires)) + 
  geom_boxplot(col = databzh$colour4) +
  coord_flip() +
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()</code></pre>
<h2><a href="http://colinfay.me/wp-content/uploads/2017/03/Rplot18.jpeg"><img class="aligncenter size-large wp-image-1397" src="http://colinfay.me/wp-content/uploads/2017/03/Rplot18-1024x512.jpeg" alt="" width="840" height="420" /></a></h2>
<h4>Dotplot</h4>
<pre class="r"><code>prim %&gt;%
  group_by(Région) %&gt;%
  summarise(somme = sum(Bénéficiaires))%&gt;%
  ggplot(aes(reorder(Région, somme), somme)) +
  geom_point(col = databzh$colour1, size=4) + 
  coord_flip() + 
  xlab("")+
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()</code></pre>
<h3><a href="http://colinfay.me/wp-content/uploads/2017/03/Rplot15.jpeg"><img class="aligncenter size-large wp-image-1399" src="http://colinfay.me/wp-content/uploads/2017/03/Rplot15-1024x512.jpeg" alt="" width="840" height="420" /></a></h3>
<h4>Dotplot bis</h4>
<pre class="r"><code>prim %&gt;%
  group_by(Établissement) %&gt;%
  summarise(somme = sum(Bénéficiaires))%&gt;%
  na.omit() %&gt;%
  ggplot(aes(reorder(Établissement, somme), somme)) +
  geom_point(col = databzh$colour3, size=4) + 
  coord_flip() + 
  xlab("")+
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()</code></pre>
<h3><a href="http://colinfay.me/wp-content/uploads/2017/03/Rplot17.jpeg"><img class="aligncenter size-large wp-image-1400" src="http://colinfay.me/wp-content/uploads/2017/03/Rplot17-1024x512.jpeg" alt="" width="840" height="420" /></a></h3>
<h4>Histogram</h4>
<pre class="r"><code>prim %&gt;%
  group_by(Établissement) %&gt;%
  summarise(somme = sum(Bénéficiaires))%&gt;%
  na.omit() %&gt;%
  ggplot(aes(somme)) +
  geom_histogram(fill = databzh$colour3) + 
  xlab("Bénéficiaire")+
  ylab("Nombre d'établissements dans la tranche") +
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()</code></pre>
<h3><a href="http://colinfay.me/wp-content/uploads/2017/03/Rplot19.jpeg"><img class="aligncenter size-large wp-image-1401" src="http://colinfay.me/wp-content/uploads/2017/03/Rplot19-1024x512.jpeg" alt="" width="840" height="420" /></a></h3>
<h4>Line plot</h4>
<pre class="r"><code>prim %&gt;%
  group_by(Année, `Groupe de corps`) %&gt;%
  summarise(somme = sum(Bénéficiaires))%&gt;%
  ggplot(aes(Année, somme, group =`Groupe de corps`, col =`Groupe de corps`)) + 
  geom_line(stat = "identity", size = 3) +
  scale_color_manual(values = databzh$colours) +
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()</code></pre>
<h3><a href="http://colinfay.me/wp-content/uploads/2017/03/Rplot14.jpeg"><img class="aligncenter size-large wp-image-1402" src="http://colinfay.me/wp-content/uploads/2017/03/Rplot14-1024x512.jpeg" alt="" width="840" height="420" /></a></h3>
<h4>Barplot</h4>
<pre class="r"><code>prim %&gt;%
  group_by(Année, Bénéficiaires, Sexe) %&gt;%
  summarise(somme = sum(Bénéficiaires))%&gt;%
  ggplot(aes(Année, somme)) + 
  geom_bar(stat = "identity", fill = databzh$colour2) +
  scale_fill_manual(values = databzh$colours) +
  facet_grid(Sexe~.) +
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()

</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2017/03/Rplot09.jpeg"><img class="aligncenter size-large wp-image-1403" src="http://colinfay.me/wp-content/uploads/2017/03/Rplot09-1024x512.jpeg" alt="" width="840" height="420" /></a>
