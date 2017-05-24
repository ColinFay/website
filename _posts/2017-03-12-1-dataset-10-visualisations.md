ID: 1388
post_title: >
  1 dataset, 10 visualisations — retour
  de La Fabrique de l’info
author: colin_fay
post_date: 2017-03-12 18:00:45
post_excerpt: ""
layout: post
permalink: >
  /1-dataset-10-visualisations/
published: true
---
<h2>Retour sur l’atelier dataviz de la Fabrique de l’info !<!--more--></h2>
<div id="la-fabrique-de-linfo" class="section level2">
<h2>La fabrique de l’info</h2>
<a href="http://colinfay.me/wp-content/uploads/2017/03/IMG_2771.jpg"><img class="size-medium wp-image-1412 alignleft" src="http://colinfay.me/wp-content/uploads/2017/03/IMG_2771-300x225.jpg" alt="" width="300" height="225" /></a>Lors de cette journée d’ateliers et de conférences, j’ai animé un atelier sur la thématique de la dataviz. Le message ? Une visualisation de données, autant qu'un texte, relève d’un choix éditorial — et un dataset peut dire énormément de choses différentes en fonction du message que l'on souhaite passer.
<p style="text-align: center;"><a href="https://github.com/ColinFay/conf/blob/master/fabrique-info/La%20fabrique%20de%20l'info.pdf">Retrouvez les slides de la présentation ici !</a></p>
Pour cela, j’ai notamment pris l’exemple d’un <a href="https://data.enseignementsup-recherche.gouv.fr/explore/dataset/fr-esr-pes-pedr-beneficiaires/">jeu de données</a>, que j'ai décliné en 10 visualisations. Voici comment les réaliser avec R.

</div>
<div id="jeu-de-donnees-10-visualisations" class="section level2">
<h2>1 jeu de données, 10 visualisations</h2>
<em>Notes
</em>Les dataviz ont été crées avec le support des <code>databzhtools</code> que vous pouvez télécharger ici : <a class="uri" href="https://github.com/DataBzh/data-bzh-tools">https://github.com/DataBzh/Data-bzh-tools</a>
Pour reproduire exactement ces visualisations, téléchargez ces outils, et charger les via : <code>source("data-bzh-tools-master/main.R")</code>
<pre class="r"><code>#Charger les données
library(tidyverse)
source(file = "data-bzh-tools-master/main.R")
prim &lt;- read_csv2("https://data.enseignementsup-recherche.gouv.fr/explore/dataset/fr-esr-pes-pedr-beneficiaires/download/?format=csv&amp;timezone=Europe/Berlin&amp;use_labels_for_header=true")
prim$Année &lt;- paste0("01-01-",prim$Année) %&gt;%
  lubridate::dmy()</code></pre>
<div id="visualisation-deffectifs" class="section level3">
<h3>Visualisation d’effectifs</h3>
<pre class="r"><code>prim %&gt;%
  group_by(Année) %&gt;%
  summarise(somme = sum(Bénéficiaires))%&gt;%
  ggplot(aes(Année, somme)) + 
  geom_bar(stat = "identity", fill = databzh$colour1) + 
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()</code></pre>
</div>
<div id="proportions-par-secteur" class="section level3">
<h3><a href="http://colinfay.me/wp-content/uploads/2017/03/Rplot06.jpeg"><img class="aligncenter size-large wp-image-1393" src="http://colinfay.me/wp-content/uploads/2017/03/Rplot06-1024x512.jpeg" alt="" width="840" height="420" /></a></h3>
<h3>Proportions par secteur</h3>
<pre class="r"><code>prim %&gt;%
  group_by(Année, `Secteur disciplinaire`) %&gt;%
  summarise(somme = sum(Bénéficiaires))%&gt;%
  ggplot(aes(Année, somme, fill = `Secteur disciplinaire`)) + 
  scale_fill_manual(values = databzh$colours) +
  geom_bar(stat = "identity", position = "fill") + 
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()</code></pre>
</div>
<div id="par-secteur-en-valeur-absolue" class="section level3">
<h3><a href="http://colinfay.me/wp-content/uploads/2017/03/Rplot07.jpeg"><img class="aligncenter size-large wp-image-1394" src="http://colinfay.me/wp-content/uploads/2017/03/Rplot07-1024x512.jpeg" alt="" width="840" height="420" /></a></h3>
<h3>Par secteur, en valeur absolue</h3>
<pre class="r"><code>prim %&gt;%
  group_by(Année, Bénéficiaires, `Secteur disciplinaire`) %&gt;%
  summarise(somme = sum(Bénéficiaires))%&gt;%
  ggplot(aes(Année,somme, group = `Secteur disciplinaire`, col = `Secteur disciplinaire`)) + 
  geom_point() +
  scale_color_manual(values = databzh$colours) +
  facet_grid(~`Secteur disciplinaire`) +
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()</code></pre>
</div>
<div id="carte-de-france" class="section level3">
<h3><a href="http://colinfay.me/wp-content/uploads/2017/03/Rplot12.jpeg"><img class="aligncenter size-large wp-image-1395" src="http://colinfay.me/wp-content/uploads/2017/03/Rplot12-1024x512.jpeg" alt="" width="840" height="420" /></a></h3>
<h3>Carte de France</h3>
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
</div>
<div id="boxplot" class="section level3">
<h3><a href="http://colinfay.me/wp-content/uploads/2017/03/Rplot20.jpeg"><img class="aligncenter size-large wp-image-1396" src="http://colinfay.me/wp-content/uploads/2017/03/Rplot20-1024x512.jpeg" alt="" width="840" height="420" /></a></h3>
<h3>Boxplot</h3>
<pre class="r"><code>ggplot(prim, aes(Région, Bénéficiaires)) + 
  geom_boxplot(col = databzh$colour4) +
  coord_flip() +
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()</code></pre>
</div>
</div>
<div id="dotplot" class="section level2">
<h2><a href="http://colinfay.me/wp-content/uploads/2017/03/Rplot18.jpeg"><img class="aligncenter size-large wp-image-1397" src="http://colinfay.me/wp-content/uploads/2017/03/Rplot18-1024x512.jpeg" alt="" width="840" height="420" /></a></h2>
<h2>Dotplot</h2>
<pre class="r"><code>prim %&gt;%
  group_by(Région) %&gt;%
  summarise(somme = sum(Bénéficiaires))%&gt;%
  ggplot(aes(reorder(Région, somme), somme)) +
  geom_point(col = databzh$colour1, size=4) + 
  coord_flip() + 
  xlab("")+
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()</code></pre>
<div id="boxplot-bis" class="section level3">
<h3><a href="http://colinfay.me/wp-content/uploads/2017/03/Rplot15.jpeg"><img class="aligncenter size-large wp-image-1399" src="http://colinfay.me/wp-content/uploads/2017/03/Rplot15-1024x512.jpeg" alt="" width="840" height="420" /></a></h3>
<h3>Dotplot bis</h3>
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
</div>
<div id="histogramme" class="section level3">
<h3><a href="http://colinfay.me/wp-content/uploads/2017/03/Rplot17.jpeg"><img class="aligncenter size-large wp-image-1400" src="http://colinfay.me/wp-content/uploads/2017/03/Rplot17-1024x512.jpeg" alt="" width="840" height="420" /></a></h3>
<h3>Histogramme</h3>
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
</div>
<div id="ligne" class="section level3">
<h3><a href="http://colinfay.me/wp-content/uploads/2017/03/Rplot19.jpeg"><img class="aligncenter size-large wp-image-1401" src="http://colinfay.me/wp-content/uploads/2017/03/Rplot19-1024x512.jpeg" alt="" width="840" height="420" /></a></h3>
<h3>Ligne</h3>
<pre class="r"><code>prim %&gt;%
  group_by(Année, `Groupe de corps`) %&gt;%
  summarise(somme = sum(Bénéficiaires))%&gt;%
  ggplot(aes(Année, somme, group =`Groupe de corps`, col =`Groupe de corps`)) + 
  geom_line(stat = "identity", size = 3) +
  scale_color_manual(values = databzh$colours) +
  labs(title = "Les bénéficiaires de la prime d'excellence scientifique") + 
  databzhTheme()</code></pre>
</div>
<div id="barplot" class="section level3">
<h3><a href="http://colinfay.me/wp-content/uploads/2017/03/Rplot14.jpeg"><img class="aligncenter size-large wp-image-1402" src="http://colinfay.me/wp-content/uploads/2017/03/Rplot14-1024x512.jpeg" alt="" width="840" height="420" /></a></h3>
<h3>Barplot</h3>
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

</div>
</div>