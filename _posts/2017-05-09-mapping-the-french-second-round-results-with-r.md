---
ID: 1731
post_title: 'Mapping the French second round results with R'
author: colin_fay
post_date: 2017-05-09 14:00:31
post_excerpt: ""
layout: single
permalink: /mapping-the-french-second-round-results-with-r/
published: true
---
## Visualising the second round results with maps made with R. <!--more-->
### The dataset
The dataset used here is available on data.gouv: <a href="https://www.data.gouv.fr/fr/datasets/election-presidentielle-des-23-avril-et-7-mai-2017-resultats-du-2eme-tour-2/" target="_blank" rel="noopener noreferrer">Election présidentielle des 23 avril et 7 mai 2017 - Résultats du 2ème tour</a>. In order to make it easier to import, I've manually converted the xls file to csv.
### Load libraries and data
Let's load this dataset, as well as the map of France available in _ggplot2_.
<pre class="r"><code class="r"><span class="keyword">library<span class="paren">(<span class="identifier">tidyverse<span class="paren">)
<span class="keyword">library<span class="paren">(<span class="identifier">stringr<span class="paren">)
<span class="keyword">library<span class="paren">(<span class="identifier">stringi<span class="paren">)
<span class="identifier">result <span class="operator"><- <span class="identifier">read_csv2<span class="paren">(<span class="string">"Presidentielle_2017_Resultats_Communes_Tour_2.csv"<span class="paren">)
<span class="identifier">map <span class="operator"><- <span class="identifier">map_data<span class="paren">(<span class="string">"france"<span class="paren">)
```
### Clean data
Before mapping the results, we need to transform and clean the data.frame ```{r} 
result
```.
<pre class="r"><code class="r"><span class="identifier">result <span class="operator"><- <span class="identifier">result <span class="operator">%>%
  <span class="identifier">group_by<span class="paren">(<span class="identifier">`Libellé du département`<span class="paren">) <span class="operator">%>%
  <span class="identifier">summarise<span class="paren">(<span class="identifier">tot_vot <span class="operator">= <span class="identifier">sum<span class="paren">(<span class="identifier">Exprimé<span class="identifier">s<span class="paren">), 
            <span class="identifier">tot_blanc <span class="operator">= <span class="identifier">sum<span class="paren">(<span class="identifier">Blancs<span class="paren">),
            <span class="identifier">pourcentage_blanc <span class="operator">= <span class="identifier">tot_blanc <span class="operator">/ <span class="identifier">sum<span class="paren">(<span class="identifier">Votants<span class="paren">) <span class="operator">* <span class="number">100, 
            <span class="identifier">tot_abs <span class="operator">= <span class="identifier">sum<span class="paren">(<span class="identifier">Abstentions<span class="paren">), 
            <span class="identifier">pourcentage_abs <span class="operator">= <span class="identifier">tot_abs <span class="operator">/ <span class="identifier">sum<span class="paren">(<span class="identifier">Inscrits<span class="paren">)<span class="operator">* <span class="number">100,
            <span class="identifier">tot_macron <span class="operator">= <span class="identifier">sum<span class="paren">(<span class="identifier">Voix<span class="paren">), 
            <span class="identifier">tot_lepen <span class="operator">= <span class="identifier">sum<span class="paren">(<span class="identifier">Voix_1<span class="paren">), 
            <span class="identifier">pourcentage_macron <span class="operator">= <span class="identifier">tot_macron <span class="operator">/ <span class="identifier">tot_vot <span class="operator">* <span class="number">100, 
            <span class="identifier">pourcentage_lepen <span class="operator">= <span class="identifier">tot_lepen <span class="operator">/ <span class="identifier">tot_vot <span class="operator">* <span class="number">100<span class="paren">) 
<span class="identifier">names<span class="paren">(<span class="identifier">result<span class="paren">)<span class="paren">[<span class="number">1<span class="paren">] <span class="operator"><- <span class="string">"region"
<span class="identifier">result<span class="operator">$<span class="identifier">region <span class="operator"><- <span class="identifier">stri_trans_general<span class="paren">(<span class="identifier">result<span class="operator">$<span class="identifier">region, <span class="string">"Latin-ASCII"<span class="paren">) <span class="operator">%>%
  <span class="identifier">str_replace_all<span class="paren">(<span class="string">"Cote-d'Or", <span class="string">"Cote-Dor"<span class="paren">) <span class="operator">%>%
  <span class="identifier">str_replace_all<span class="paren">(<span class="string">"Cotes-d'Armor", <span class="string">"Cotes-Darmor"<span class="paren">) <span class="operator">%>%
  <span class="identifier">str_replace_all<span class="paren">(<span class="string">"Corse-du-Sud", <span class="string">"Corse du Sud"<span class="paren">) <span class="operator">%>%
  <span class="identifier">str_replace_all<span class="paren">(<span class="string">"Val-d'Oise", <span class="string">"Val-Doise"<span class="paren">) <span class="operator">%>%
  <span class="identifier">str_replace_all<span class="paren">(<span class="string">"Corse-du-Sud", <span class="string">"Corse du Sud"<span class="paren">)
```
We now got a table containing the key figures by department. The name of the first column has been modified, in order to match the `region` name of the `map` table. The character replacement sequence is due to the English notation in `map`, and was required in order to join the tables properly.
<pre class="r"><code class="r"><span class="identifier">result_map <span class="operator"><- <span class="identifier">left_join<span class="paren">(<span class="identifier">x <span class="operator">= <span class="identifier">map<span class="paren">[,<span class="operator">-<span class="number">6<span class="paren">], <span class="identifier">y <span class="operator">= <span class="identifier">result<span class="paren">)
```
### Visualisation
Let's now project our variables on maps. You need to play with the ```{r} 
scale_fill_
``` argument to manage the color scheme used on each card.
<p style="text-align: right;">_Note : this article was first published in french. _
_I've kept the original plot titles._</p>

<pre class="r"><code class="r"><span class="identifier">map_theme <span class="operator"><- <span class="identifier">theme<span class="paren">(<span class="identifier">title<span class="operator">=<span class="identifier">element_text<span class="paren">(<span class="paren">),
                   <span class="identifier">plot.title<span class="operator">=<span class="identifier">element_text<span class="paren">(<span class="identifier">margin<span class="operator">=<span class="identifier">margin<span class="paren">(<span class="number">20,<span class="number">20,<span class="number">20,<span class="number">20<span class="paren">), <span class="identifier">size<span class="operator">=<span class="number">18, <span class="identifier">hjust <span class="operator">= <span class="number">0.5<span class="paren">),
                   <span class="identifier">axis.text.x<span class="operator">=<span class="identifier">element_blank<span class="paren">(<span class="paren">),
                   <span class="identifier">axis.text.y<span class="operator">=<span class="identifier">element_blank<span class="paren">(<span class="paren">),
                   <span class="identifier">axis.ticks<span class="operator">=<span class="identifier">element_blank<span class="paren">(<span class="paren">),
                   <span class="identifier">axis.title.x<span class="operator">=<span class="identifier">element_blank<span class="paren">(<span class="paren">),
                   <span class="identifier">axis.title.y<span class="operator">=<span class="identifier">element_blank<span class="paren">(<span class="paren">),
                   <span class="identifier">panel.grid.major<span class="operator">= <span class="identifier">element_blank<span class="paren">(<span class="paren">), 
                   <span class="identifier">panel.background<span class="operator">= <span class="identifier">element_blank<span class="paren">(<span class="paren">)<span class="paren">) 

<span class="identifier">ggplot<span class="paren">(<span class="identifier">result_map, <span class="identifier">aes<span class="paren">(<span class="identifier">long,<span class="identifier">lat, <span class="identifier">group <span class="operator">= <span class="identifier">group, <span class="identifier">fill <span class="operator">= <span class="identifier">pourcentage_blanc<span class="paren">)<span class="paren">) <span class="operator">+
  <span class="identifier">geom_polygon<span class="paren">(<span class="paren">) <span class="operator">+
  <span class="identifier">coord_map<span class="paren">(<span class="paren">) <span class="operator">+
  <span class="identifier">scale_fill_gradient<span class="paren">(<span class="identifier">name <span class="operator">= <span class="string">"Pourcentage votes blancs"<span class="paren">) <span class="operator">+
  <span class="identifier">labs<span class="paren">(<span class="identifier">x <span class="operator">= <span class="string">"", 
       <span class="identifier">y <span class="operator">= <span class="string">"", 
       <span class="identifier">title <span class="operator">= <span class="string">"Pourcentage de votes blancs au second tour des présidentielles 2017", 
       <span class="identifier">subtitle <span class="operator">= <span class="string">"Données via data.gouv",
       <span class="identifier">caption <span class="operator">= <span class="string">"http://colinfay.me"<span class="paren">) <span class="operator">+
  <span class="identifier">map_theme

```
<a href="https://colinfay.github.io/wp-content/uploads/2017/05/second-tour-blanc.png"><img class="aligncenter size-full wp-image-1716" src="https://colinfay.github.io/wp-content/uploads/2017/05/second-tour-blanc.png" alt="blancs du second tour" width="1000" height="500" /></a>
<pre class="r"><code class="r"><span class="identifier">ggplot<span class="paren">(<span class="identifier">result_map, <span class="identifier">aes<span class="paren">(<span class="identifier">long,<span class="identifier">lat, <span class="identifier">group <span class="operator">= <span class="identifier">group, <span class="identifier">fill <span class="operator">= <span class="identifier">pourcentage_abs<span class="paren">)<span class="paren">) <span class="operator">+
  <span class="identifier">geom_polygon<span class="paren">(<span class="paren">) <span class="operator">+
  <span class="identifier">coord_map<span class="paren">(<span class="paren">) <span class="operator">+
  <span class="identifier">scale_fill_gradient2<span class="paren">(<span class="identifier">name <span class="operator">= <span class="string">"Pourcentage Abstention"<span class="paren">) <span class="operator">+
  <span class="identifier">labs<span class="paren">(<span class="identifier">x <span class="operator">= <span class="string">"", 
       <span class="identifier">y <span class="operator">= <span class="string">"", 
       <span class="identifier">title <span class="operator">= <span class="string">"Pourcentage d'abstention au second tour des présidentielles 2017", 
       <span class="identifier">subtitle <span class="operator">= <span class="string">"Données via data.gouv",
       <span class="identifier">caption <span class="operator">= <span class="string">"http://colinfay.me"<span class="paren">) <span class="operator">+
  <span class="identifier">map_theme 
```
<a href="https://colinfay.github.io/wp-content/uploads/2017/05/abstention-second-tour.png"><img class="aligncenter size-full wp-image-1717" src="https://colinfay.github.io/wp-content/uploads/2017/05/abstention-second-tour.png" alt="abstention second tour" width="1000" height="500" /></a>
<pre class="r"><code class="r"><span class="identifier">ggplot<span class="paren">(<span class="identifier">result_map, <span class="identifier">aes<span class="paren">(<span class="identifier">long,<span class="identifier">lat, <span class="identifier">group <span class="operator">= <span class="identifier">group, <span class="identifier">fill <span class="operator">= <span class="identifier">pourcentage_macron<span class="paren">)<span class="paren">) <span class="operator">+
  <span class="identifier">geom_polygon<span class="paren">(<span class="paren">) <span class="operator">+
  <span class="identifier">coord_map<span class="paren">(<span class="paren">) <span class="operator">+
  <span class="identifier">scale_fill_gradientn<span class="paren">(<span class="identifier">colours <span class="operator">= <span class="identifier">c<span class="paren">(<span class="string">"yellow",<span class="string">"red"<span class="paren">), <span class="identifier">name <span class="operator">= <span class="string">"Pourcentage E. Macron"<span class="paren">) <span class="operator">+
  <span class="identifier">labs<span class="paren">(<span class="identifier">x <span class="operator">= <span class="string">"", 
       <span class="identifier">y <span class="operator">= <span class="string">"", 
       <span class="identifier">title <span class="operator">= <span class="string">"Résultats de E. Macron au second tour des présidentielles 2017", 
       <span class="identifier">subtitle <span class="operator">= <span class="string">"Données via data.gouv",
       <span class="identifier">caption <span class="operator">= <span class="string">"http://colinfay.me"<span class="paren">) <span class="operator">+
  <span class="identifier">map_theme 
```
<a href="https://colinfay.github.io/wp-content/uploads/2017/05/macron.png"><img class="aligncenter size-full wp-image-1725" src="https://colinfay.github.io/wp-content/uploads/2017/05/macron.png" alt="macron second tour" width="1000" height="500" /></a>
<pre class="r"><code class="r"><span class="identifier">
ggplot<span class="paren">(<span class="identifier">result_map, <span class="identifier">aes<span class="paren">(<span class="identifier">long,<span class="identifier">lat, <span class="identifier">group <span class="operator">= <span class="identifier">group, <span class="identifier">fill <span class="operator">= <span class="identifier">pourcentage_lepen<span class="paren">)<span class="paren">) <span class="operator">+
  <span class="identifier">geom_polygon<span class="paren">(<span class="paren">) <span class="operator">+
  <span class="identifier">coord_map<span class="paren">(<span class="paren">) <span class="operator">+
  <span class="identifier">scale_fill_gradientn<span class="paren">(<span class="identifier">colours <span class="operator">= <span class="identifier">terrain.colors<span class="paren">(<span class="number">10<span class="paren">), <span class="identifier">name <span class="operator">= <span class="string">"Pourcentage M. Le Pen"<span class="paren">) <span class="operator">+
  <span class="identifier">labs<span class="paren">(<span class="identifier">x <span class="operator">= <span class="string">"", 
       <span class="identifier">y <span class="operator">= <span class="string">"", 
       <span class="identifier">title <span class="operator">= <span class="string">"Résultats de M. Le Pen au second tour des présidentielles 2017", 
       <span class="identifier">subtitle <span class="operator">= <span class="string">"Données via data.gouv",
       <span class="identifier">caption <span class="operator">= <span class="string">"http://colinfay.me"<span class="paren">) <span class="operator">+
  <span class="identifier">map_theme 
```
<a href="https://colinfay.github.io/wp-content/uploads/2017/05/mlp.png"><img class="aligncenter size-full wp-image-1724" src="https://colinfay.github.io/wp-content/uploads/2017/05/mlp.png" alt="Votes pour Marine Le Pen au second tour" width="1000" height="500" /></a>
