---
ID: 1433
post_title: Une heatmap avec R et ggplot2
author: colin_fay
post_date: 2017-03-19 20:28:24
post_excerpt: ""
layout: post
permalink: >
  /heatmap-avec-r-et-ggplot2/
published: true
---
<h2>Un court tutorial sur les heatmaps avec R, inspirés d'articles sur databzh.</h2>
<!--more-->

L'idée de cet article prend racine dans deux billets sur Data Bzh :
- <a href="http://data-bzh.fr/trafic-web-site-rennes-metropole-2016/">Trafic du site web de Rennes Metropole en 2016</a>
- <a href="http://data-bzh.fr/prenoms-bretagne-1900-aujourdhui/">Les prénoms en Bretagne, de 1900 à aujourd'hui</a>

Dans ce court post, retrouvez le déroulement de la création d'une heatmap d'un prénom par année et par département. Le jeu de données est disponible sur <a href="https://www.data.gouv.fr/fr/datasets/fichier-des-prenoms-edition-2016/">data.gouv</a>. Il a été téléchargé et unzippé en dehors de R.
<div id="loading" class="section level2">
<h2>Loading</h2>
<pre class="r"><code>library(tidyverse)</code></pre>
<pre><code>## Loading tidyverse: ggplot2
## Loading tidyverse: tibble
## Loading tidyverse: tidyr
## Loading tidyverse: readr
## Loading tidyverse: purrr
## Loading tidyverse: dplyr</code></pre>
<pre><code></code><code></code><code>name &lt;- read.table("/home/colin/Téléchargements/dpt2015.txt", stringsAsFactors = FALSE, sep = "\t", encoding = "latin1", header = TRUE, col.names = c("sexe","prenom","annee","dpt","nombre")) %&gt;%
  na.omit()</code></pre>
<pre><code></code><code>name$annee &lt;- as.Date(name$annee, "%Y")</code></pre>
Nous avons maintenant un jeu de données propre, avec les noms et les départements.
<div id="heatmap" class="section level3">
<h3>Heatmap</h3>
Une heatmap se crée le geom <code>geom_tile</code> de <code>ggplot2</code>. Voici sa construction étape par étape.
<pre class="r"><code>choix &lt;- "COLIN"
name %&gt;%
  #Filtre par nom
  filter(prenom == choix) %&gt;%
  
  #Groupe par année et département
  group_by(annee, dpt) %&gt;%
  
  #Résumé de l'effectif par année et département
  summarise(somme = sum(nombre)) %&gt;%
  
  #Suppression des NA
  na.omit() %&gt;% 
  
  #Initialisation de ggplot
  ggplot(aes(annee, dpt, fill = somme)) +
  
  #THE MAN OF THE HOUR
  geom_tile() +
  
  #Échelle
  scale_x_date(limits =  c(lubridate::ymd("1900-01-01"), lubridate::ymd("2015-01-01"))) +
  
  #Et deux trois paillettes pour rendre le tout plus joli
  xlab("Année") +
  ylab("Département") +
  labs(title = paste0("Apparition du prénom ", tolower(choix)," par département, 1900-2015")) + 
  theme_minimal()</code></pre>
</div>
<a href="http://colinfay.me/wp-content/uploads/2017/03/names-colin.png"><img class="aligncenter size-full wp-image-1587" src="http://colinfay.me/wp-content/uploads/2017/03/names-colin.png" alt="Colin par département" width="1000" height="500" /></a>

Oui, c'est aussi simple que ça. Essayons avec un autre prénom.

(Bien sûr, vous pouvez choisir une autre échelle pour les couleurs.)
<pre class="r"><code>choix &lt;- "ELISABETH"
name %&gt;%
  filter(prenom == choix) %&gt;%
  group_by(annee, dpt) %&gt;%
  summarise(somme = sum(nombre)) %&gt;%
  na.omit() %&gt;% 
  ggplot(aes(annee, dpt, fill = somme)) +
  geom_tile() +
  scale_x_date(limits =  c(lubridate::ymd("1900-01-01"), lubridate::ymd("2015-01-01"))) +
  #Changer l'échelle de couleurs
  scale_fill_gradient(low = "#E18C8C", high = "#973232") +
  xlab("Année") +
  ylab("Département") +
  labs(title = paste0("Apparition du prénom ", tolower(choix)," par département, 1900-2015")) + 
  theme_minimal()
</code></pre>
<a href="http://colinfay.me/wp-content/uploads/2017/03/prenom-elisabeth-rstats.png"><img class="aligncenter size-full wp-image-1589" src="http://colinfay.me/wp-content/uploads/2017/03/prenom-elisabeth-rstats.png" alt="Elisabeth prénom" width="1000" height="500" /></a>

Simple, n'est-ce pas !

</div>