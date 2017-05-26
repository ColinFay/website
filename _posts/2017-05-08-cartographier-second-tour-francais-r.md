---
ID: 1705
post_title: 'Cartographier le second tour français avec R'
author: colin_fay
post_date: 2017-05-08 18:08:31
post_excerpt: ""
layout: single
permalink: /cartographier-second-tour-francais-r/
published: true
---
## Aperçu du vote au second tour en France, via des cartes réalisées avec R. <!--more-->
### Le jeu de données
Le dataset utilisé ici est disponible sur data.gouv : <a href="https://www.data.gouv.fr/fr/datasets/election-presidentielle-des-23-avril-et-7-mai-2017-resultats-du-2eme-tour-2/" target="_blank" rel="noopener noreferrer">Election présidentielle des 23 avril et 7 mai 2017 - Résultats du 2ème tour</a>. Pour une meilleur compatibilité, j'ai manuellement converti le fichier xls en csv.
### Charger les librairies et les données
Chargeons ce jeu de données, ainsi que la carte de France disponible nativement avec ggplot2.
<pre class="r"><code class="r"><span class="keyword">library<span class="paren">(<span class="identifier">tidyverse<span class="paren">)
<span class="keyword">library<span class="paren">(<span class="identifier">stringr<span class="paren">)
<span class="keyword">library<span class="paren">(<span class="identifier">stringi<span class="paren">)
<span class="identifier">result <span class="operator">&lt;- <span class="identifier">read_csv2<span class="paren">(<span class="string">"Presidentielle_2017_Resultats_Communes_Tour_2.csv"<span class="paren">)
<span class="identifier">map <span class="operator">&lt;- <span class="identifier">map_data<span class="paren">(<span class="string">"france"<span class="paren">)
```
### Nettoyage des données

Avant de représenter les résultats sur une carte, nous devons commencer par transformer et nettoyer le data.frame <code>result</code>.
<pre class="r"><code class="r"><span class="identifier">result <span class="operator">&lt;- <span class="identifier">result <span class="operator">%&gt;%
  <span class="identifier">group_by<span class="paren">(<span class="identifier">`Libellé du département`<span class="paren">) <span class="operator">%&gt;%
  <span class="identifier">summarise<span class="paren">(<span class="identifier">tot_vot <span class="operator">= <span class="identifier">sum<span class="paren">(<span class="identifier">Exprimé<span class="identifier">s<span class="paren">), 
            <span class="identifier">tot_blanc <span class="operator">= <span class="identifier">sum<span class="paren">(<span class="identifier">Blancs<span class="paren">),
            <span class="identifier">pourcentage_blanc <span class="operator">= <span class="identifier">tot_blanc <span class="operator">/ <span class="identifier">sum<span class="paren">(<span class="identifier">Votants<span class="paren">) <span class="operator">* <span class="number">100, 
            <span class="identifier">tot_abs <span class="operator">= <span class="identifier">sum<span class="paren">(<span class="identifier">Abstentions<span class="paren">), 
            <span class="identifier">pourcentage_abs <span class="operator">= <span class="identifier">tot_abs <span class="operator">/ <span class="identifier">sum<span class="paren">(<span class="identifier">Inscrits<span class="paren">)<span class="operator">* <span class="number">100,
            <span class="identifier">tot_macron <span class="operator">= <span class="identifier">sum<span class="paren">(<span class="identifier">Voix<span class="paren">), 
            <span class="identifier">tot_lepen <span class="operator">= <span class="identifier">sum<span class="paren">(<span class="identifier">Voix_1<span class="paren">), 
            <span class="identifier">pourcentage_macron <span class="operator">= <span class="identifier">tot_macron <span class="operator">/ <span class="identifier">tot_vot <span class="operator">* <span class="number">100, 
            <span class="identifier">pourcentage_lepen <span class="operator">= <span class="identifier">tot_lepen <span class="operator">/ <span class="identifier">tot_vot <span class="operator">* <span class="number">100<span class="paren">) 
<span class="identifier">names<span class="paren">(<span class="identifier">result<span class="paren">)<span class="paren">[<span class="number">1<span class="paren">] <span class="operator">&lt;- <span class="string">"region"
<span class="identifier">result<span class="operator">$<span class="identifier">region <span class="operator">&lt;- <span class="identifier">stri_trans_general<span class="paren">(<span class="identifier">result<span class="operator">$<span class="identifier">region, <span class="string">"Latin-ASCII"<span class="paren">) <span class="operator">%&gt;%
  <span class="identifier">str_replace_all<span class="paren">(<span class="string">"Cote-d'Or", <span class="string">"Cote-Dor"<span class="paren">) <span class="operator">%&gt;%
  <span class="identifier">str_replace_all<span class="paren">(<span class="string">"Cotes-d'Armor", <span class="string">"Cotes-Darmor"<span class="paren">) <span class="operator">%&gt;%
  <span class="identifier">str_replace_all<span class="paren">(<span class="string">"Corse-du-Sud", <span class="string">"Corse du Sud"<span class="paren">) <span class="operator">%&gt;%
  <span class="identifier">str_replace_all<span class="paren">(<span class="string">"Val-d'Oise", <span class="string">"Val-Doise"<span class="paren">) <span class="operator">%&gt;%
  <span class="identifier">str_replace_all<span class="paren">(<span class="string">"Corse-du-Sud", <span class="string">"Corse du Sud"<span class="paren">)
```
Nous voici avec un tableau contenant les chiffres clés par département, obtenu à partir des résultats par commune. Le nom de la première colonne a été modifié, afin de coller à l'étiquetage `region` du tableau `map`. La suite de remplacement de caractères est due à la notation anglaise de `map` — une transformation a été indispensable pour effectuer la jointure correctement.
<pre class="r"><code class="r"><span class="identifier">result_map <span class="operator">&lt;- <span class="identifier">left_join<span class="paren">(<span class="identifier">x <span class="operator">= <span class="identifier">map<span class="paren">[,<span class="operator">-<span class="number">6<span class="paren">], <span class="identifier">y <span class="operator">= <span class="identifier">result<span class="paren">)
```
### Visualisation
Projetons maintenant nos différentes variables avec R. Ici, c'est l'argument `scale_fill_` qui va gérer l'échelle de couleurs utilisée pour chaque carte.
<pre class="r"><code class="r"><span class="identifier">map_theme <span class="operator">&lt;- <span class="identifier">theme<span class="paren">(<span class="identifier">title<span class="operator">=<span class="identifier">element_text<span class="paren">(<span class="paren">),
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
