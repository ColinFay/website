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
<h2>Aperçu du vote au second tour en France, via des cartes réalisées avec R. <!--more--></h2>
<h3>Le jeu de données</h3>
Le dataset utilisé ici est disponible sur data.gouv : <a href="https://www.data.gouv.fr/fr/datasets/election-presidentielle-des-23-avril-et-7-mai-2017-resultats-du-2eme-tour-2/" target="_blank" rel="noopener noreferrer">Election présidentielle des 23 avril et 7 mai 2017 - Résultats du 2ème tour</a>. Pour une meilleur compatibilité, j'ai manuellement converti le fichier xls en csv.
<h3>Charger les librairies et les données</h3>
Chargeons ce jeu de données, ainsi que la carte de France disponible nativement avec ggplot2.
<pre class="r"><code class="r"><span class="keyword">library</span><span class="paren">(</span><span class="identifier">tidyverse</span><span class="paren">)</span>
<span class="keyword">library</span><span class="paren">(</span><span class="identifier">stringr</span><span class="paren">)</span>
<span class="keyword">library</span><span class="paren">(</span><span class="identifier">stringi</span><span class="paren">)</span>
<span class="identifier">result</span> <span class="operator">&lt;-</span> <span class="identifier">read_csv2</span><span class="paren">(</span><span class="string">"Presidentielle_2017_Resultats_Communes_Tour_2.csv"</span><span class="paren">)</span>
<span class="identifier">map</span> <span class="operator">&lt;-</span> <span class="identifier">map_data</span><span class="paren">(</span><span class="string">"france"</span><span class="paren">)</span></code></pre>
### Nettoyage des données

Avant de représenter les résultats sur une carte, nous devons commencer par transformer et nettoyer le data.frame <code>result</code>.
<pre class="r"><code class="r"><span class="identifier">result</span> <span class="operator">&lt;-</span> <span class="identifier">result</span> <span class="operator">%&gt;%</span>
  <span class="identifier">group_by</span><span class="paren">(</span><span class="identifier">`Libellé du département`</span><span class="paren">)</span> <span class="operator">%&gt;%</span>
  <span class="identifier">summarise</span><span class="paren">(</span><span class="identifier">tot_vot</span> <span class="operator">=</span> <span class="identifier">sum</span><span class="paren">(</span><span class="identifier">Exprim</span>é<span class="identifier">s</span><span class="paren">)</span>, 
            <span class="identifier">tot_blanc</span> <span class="operator">=</span> <span class="identifier">sum</span><span class="paren">(</span><span class="identifier">Blancs</span><span class="paren">)</span>,
            <span class="identifier">pourcentage_blanc</span> <span class="operator">=</span> <span class="identifier">tot_blanc</span> <span class="operator">/</span> <span class="identifier">sum</span><span class="paren">(</span><span class="identifier">Votants</span><span class="paren">)</span> <span class="operator">*</span> <span class="number">100</span>, 
            <span class="identifier">tot_abs</span> <span class="operator">=</span> <span class="identifier">sum</span><span class="paren">(</span><span class="identifier">Abstentions</span><span class="paren">)</span>, 
            <span class="identifier">pourcentage_abs</span> <span class="operator">=</span> <span class="identifier">tot_abs</span> <span class="operator">/</span> <span class="identifier">sum</span><span class="paren">(</span><span class="identifier">Inscrits</span><span class="paren">)</span><span class="operator">*</span> <span class="number">100</span>,
            <span class="identifier">tot_macron</span> <span class="operator">=</span> <span class="identifier">sum</span><span class="paren">(</span><span class="identifier">Voix</span><span class="paren">)</span>, 
            <span class="identifier">tot_lepen</span> <span class="operator">=</span> <span class="identifier">sum</span><span class="paren">(</span><span class="identifier">Voix_1</span><span class="paren">)</span>, 
            <span class="identifier">pourcentage_macron</span> <span class="operator">=</span> <span class="identifier">tot_macron</span> <span class="operator">/</span> <span class="identifier">tot_vot</span> <span class="operator">*</span> <span class="number">100</span>, 
            <span class="identifier">pourcentage_lepen</span> <span class="operator">=</span> <span class="identifier">tot_lepen</span> <span class="operator">/</span> <span class="identifier">tot_vot</span> <span class="operator">*</span> <span class="number">100</span><span class="paren">)</span> 
<span class="identifier">names</span><span class="paren">(</span><span class="identifier">result</span><span class="paren">)</span><span class="paren">[</span><span class="number">1</span><span class="paren">]</span> <span class="operator">&lt;-</span> <span class="string">"region"</span>
<span class="identifier">result</span><span class="operator">$</span><span class="identifier">region</span> <span class="operator">&lt;-</span> <span class="identifier">stri_trans_general</span><span class="paren">(</span><span class="identifier">result</span><span class="operator">$</span><span class="identifier">region</span>, <span class="string">"Latin-ASCII"</span><span class="paren">)</span> <span class="operator">%&gt;%</span>
  <span class="identifier">str_replace_all</span><span class="paren">(</span><span class="string">"Cote-d'Or"</span>, <span class="string">"Cote-Dor"</span><span class="paren">)</span> <span class="operator">%&gt;%</span>
  <span class="identifier">str_replace_all</span><span class="paren">(</span><span class="string">"Cotes-d'Armor"</span>, <span class="string">"Cotes-Darmor"</span><span class="paren">)</span> <span class="operator">%&gt;%</span>
  <span class="identifier">str_replace_all</span><span class="paren">(</span><span class="string">"Corse-du-Sud"</span>, <span class="string">"Corse du Sud"</span><span class="paren">)</span> <span class="operator">%&gt;%</span>
  <span class="identifier">str_replace_all</span><span class="paren">(</span><span class="string">"Val-d'Oise"</span>, <span class="string">"Val-Doise"</span><span class="paren">)</span> <span class="operator">%&gt;%</span>
  <span class="identifier">str_replace_all</span><span class="paren">(</span><span class="string">"Corse-du-Sud"</span>, <span class="string">"Corse du Sud"</span><span class="paren">)</span></code></pre>
Nous voici avec un tableau contenant les chiffres clés par département, obtenu à partir des résultats par commune. Le nom de la première colonne a été modifié, afin de coller à l'étiquetage `region` du tableau `map`. La suite de remplacement de caractères est due à la notation anglaise de `map` — une transformation a été indispensable pour effectuer la jointure correctement.
<pre class="r"><code class="r"><span class="identifier">result_map</span> <span class="operator">&lt;-</span> <span class="identifier">left_join</span><span class="paren">(</span><span class="identifier">x</span> <span class="operator">=</span> <span class="identifier">map</span><span class="paren">[</span>,<span class="operator">-</span><span class="number">6</span><span class="paren">]</span>, <span class="identifier">y</span> <span class="operator">=</span> <span class="identifier">result</span><span class="paren">)</span></code></pre>
<h3>Visualisation</h3>
Projetons maintenant nos différentes variables avec R. Ici, c'est l'argument `scale_fill_` qui va gérer l'échelle de couleurs utilisée pour chaque carte.
<pre class="r"><code class="r"><span class="identifier">map_theme</span> <span class="operator">&lt;-</span> <span class="identifier">theme</span><span class="paren">(</span><span class="identifier">title</span><span class="operator">=</span><span class="identifier">element_text</span><span class="paren">(</span><span class="paren">)</span>,
                   <span class="identifier">plot.title</span><span class="operator">=</span><span class="identifier">element_text</span><span class="paren">(</span><span class="identifier">margin</span><span class="operator">=</span><span class="identifier">margin</span><span class="paren">(</span><span class="number">20</span>,<span class="number">20</span>,<span class="number">20</span>,<span class="number">20</span><span class="paren">)</span>, <span class="identifier">size</span><span class="operator">=</span><span class="number">18</span>, <span class="identifier">hjust</span> <span class="operator">=</span> <span class="number">0.5</span><span class="paren">)</span>,
                   <span class="identifier">axis.text.x</span><span class="operator">=</span><span class="identifier">element_blank</span><span class="paren">(</span><span class="paren">)</span>,
                   <span class="identifier">axis.text.y</span><span class="operator">=</span><span class="identifier">element_blank</span><span class="paren">(</span><span class="paren">)</span>,
                   <span class="identifier">axis.ticks</span><span class="operator">=</span><span class="identifier">element_blank</span><span class="paren">(</span><span class="paren">)</span>,
                   <span class="identifier">axis.title.x</span><span class="operator">=</span><span class="identifier">element_blank</span><span class="paren">(</span><span class="paren">)</span>,
                   <span class="identifier">axis.title.y</span><span class="operator">=</span><span class="identifier">element_blank</span><span class="paren">(</span><span class="paren">)</span>,
                   <span class="identifier">panel.grid.major</span><span class="operator">=</span> <span class="identifier">element_blank</span><span class="paren">(</span><span class="paren">)</span>, 
                   <span class="identifier">panel.background</span><span class="operator">=</span> <span class="identifier">element_blank</span><span class="paren">(</span><span class="paren">)</span><span class="paren">)</span> 

<span class="identifier">ggplot</span><span class="paren">(</span><span class="identifier">result_map</span>, <span class="identifier">aes</span><span class="paren">(</span><span class="identifier">long</span>,<span class="identifier">lat</span>, <span class="identifier">group</span> <span class="operator">=</span> <span class="identifier">group</span>, <span class="identifier">fill</span> <span class="operator">=</span> <span class="identifier">pourcentage_blanc</span><span class="paren">)</span><span class="paren">)</span> <span class="operator">+</span>
  <span class="identifier">geom_polygon</span><span class="paren">(</span><span class="paren">)</span> <span class="operator">+</span>
  <span class="identifier">coord_map</span><span class="paren">(</span><span class="paren">)</span> <span class="operator">+</span>
  <span class="identifier">scale_fill_gradient</span><span class="paren">(</span><span class="identifier">name</span> <span class="operator">=</span> <span class="string">"Pourcentage votes blancs"</span><span class="paren">)</span> <span class="operator">+</span>
  <span class="identifier">labs</span><span class="paren">(</span><span class="identifier">x</span> <span class="operator">=</span> <span class="string">""</span>, 
       <span class="identifier">y</span> <span class="operator">=</span> <span class="string">""</span>, 
       <span class="identifier">title</span> <span class="operator">=</span> <span class="string">"Pourcentage de votes blancs au second tour des présidentielles 2017"</span>, 
       <span class="identifier">subtitle</span> <span class="operator">=</span> <span class="string">"Données via data.gouv"</span>,
       <span class="identifier">caption</span> <span class="operator">=</span> <span class="string">"http://colinfay.me"</span><span class="paren">)</span> <span class="operator">+</span>
  <span class="identifier">map_theme
</span></code></pre>
<a href="http://colinfay.me/wp-content/uploads/2017/05/second-tour-blanc.png"><img class="aligncenter size-full wp-image-1716" src="http://colinfay.me/wp-content/uploads/2017/05/second-tour-blanc.png" alt="blancs du second tour" width="1000" height="500" /></a>
<pre class="r"><code class="r"><span class="identifier">ggplot</span><span class="paren">(</span><span class="identifier">result_map</span>, <span class="identifier">aes</span><span class="paren">(</span><span class="identifier">long</span>,<span class="identifier">lat</span>, <span class="identifier">group</span> <span class="operator">=</span> <span class="identifier">group</span>, <span class="identifier">fill</span> <span class="operator">=</span> <span class="identifier">pourcentage_abs</span><span class="paren">)</span><span class="paren">)</span> <span class="operator">+</span>
  <span class="identifier">geom_polygon</span><span class="paren">(</span><span class="paren">)</span> <span class="operator">+</span>
  <span class="identifier">coord_map</span><span class="paren">(</span><span class="paren">)</span> <span class="operator">+</span>
  <span class="identifier">scale_fill_gradient2</span><span class="paren">(</span><span class="identifier">name</span> <span class="operator">=</span> <span class="string">"Pourcentage Abstention"</span><span class="paren">)</span> <span class="operator">+</span>
  <span class="identifier">labs</span><span class="paren">(</span><span class="identifier">x</span> <span class="operator">=</span> <span class="string">""</span>, 
       <span class="identifier">y</span> <span class="operator">=</span> <span class="string">""</span>, 
       <span class="identifier">title</span> <span class="operator">=</span> <span class="string">"Pourcentage d'abstention au second tour des présidentielles 2017"</span>, 
       <span class="identifier">subtitle</span> <span class="operator">=</span> <span class="string">"Données via data.gouv"</span>,
       <span class="identifier">caption</span> <span class="operator">=</span> <span class="string">"http://colinfay.me"</span><span class="paren">)</span> <span class="operator">+</span>
  <span class="identifier">map_theme</span> </code></pre>
<a href="http://colinfay.me/wp-content/uploads/2017/05/abstention-second-tour.png"><img class="aligncenter size-full wp-image-1717" src="http://colinfay.me/wp-content/uploads/2017/05/abstention-second-tour.png" alt="abstention second tour" width="1000" height="500" /></a>
<pre class="r"><code class="r"><span class="identifier">ggplot</span><span class="paren">(</span><span class="identifier">result_map</span>, <span class="identifier">aes</span><span class="paren">(</span><span class="identifier">long</span>,<span class="identifier">lat</span>, <span class="identifier">group</span> <span class="operator">=</span> <span class="identifier">group</span>, <span class="identifier">fill</span> <span class="operator">=</span> <span class="identifier">pourcentage_macron</span><span class="paren">)</span><span class="paren">)</span> <span class="operator">+</span>
  <span class="identifier">geom_polygon</span><span class="paren">(</span><span class="paren">)</span> <span class="operator">+</span>
  <span class="identifier">coord_map</span><span class="paren">(</span><span class="paren">)</span> <span class="operator">+</span>
  <span class="identifier">scale_fill_gradientn</span><span class="paren">(</span><span class="identifier">colours</span> <span class="operator">=</span> <span class="identifier">c</span><span class="paren">(</span><span class="string">"yellow"</span>,<span class="string">"red"</span><span class="paren">)</span>, <span class="identifier">name</span> <span class="operator">=</span> <span class="string">"Pourcentage E. Macron"</span><span class="paren">)</span> <span class="operator">+</span>
  <span class="identifier">labs</span><span class="paren">(</span><span class="identifier">x</span> <span class="operator">=</span> <span class="string">""</span>, 
       <span class="identifier">y</span> <span class="operator">=</span> <span class="string">""</span>, 
       <span class="identifier">title</span> <span class="operator">=</span> <span class="string">"Résultats de E. Macron au second tour des présidentielles 2017"</span>, 
       <span class="identifier">subtitle</span> <span class="operator">=</span> <span class="string">"Données via data.gouv"</span>,
       <span class="identifier">caption</span> <span class="operator">=</span> <span class="string">"http://colinfay.me"</span><span class="paren">)</span> <span class="operator">+</span>
  <span class="identifier">map_theme</span> </code></pre>
<a href="http://colinfay.me/wp-content/uploads/2017/05/macron.png"><img class="aligncenter size-full wp-image-1725" src="http://colinfay.me/wp-content/uploads/2017/05/macron.png" alt="macron second tour" width="1000" height="500" /></a>
<pre class="r"><code class="r"><span class="identifier">
ggplot</span><span class="paren">(</span><span class="identifier">result_map</span>, <span class="identifier">aes</span><span class="paren">(</span><span class="identifier">long</span>,<span class="identifier">lat</span>, <span class="identifier">group</span> <span class="operator">=</span> <span class="identifier">group</span>, <span class="identifier">fill</span> <span class="operator">=</span> <span class="identifier">pourcentage_lepen</span><span class="paren">)</span><span class="paren">)</span> <span class="operator">+</span>
  <span class="identifier">geom_polygon</span><span class="paren">(</span><span class="paren">)</span> <span class="operator">+</span>
  <span class="identifier">coord_map</span><span class="paren">(</span><span class="paren">)</span> <span class="operator">+</span>
  <span class="identifier">scale_fill_gradientn</span><span class="paren">(</span><span class="identifier">colours</span> <span class="operator">=</span> <span class="identifier">terrain.colors</span><span class="paren">(</span><span class="number">10</span><span class="paren">)</span>, <span class="identifier">name</span> <span class="operator">=</span> <span class="string">"Pourcentage M. Le Pen"</span><span class="paren">)</span> <span class="operator">+</span>
  <span class="identifier">labs</span><span class="paren">(</span><span class="identifier">x</span> <span class="operator">=</span> <span class="string">""</span>, 
       <span class="identifier">y</span> <span class="operator">=</span> <span class="string">""</span>, 
       <span class="identifier">title</span> <span class="operator">=</span> <span class="string">"Résultats de M. Le Pen au second tour des présidentielles 2017"</span>, 
       <span class="identifier">subtitle</span> <span class="operator">=</span> <span class="string">"Données via data.gouv"</span>,
       <span class="identifier">caption</span> <span class="operator">=</span> <span class="string">"http://colinfay.me"</span><span class="paren">)</span> <span class="operator">+</span>
  <span class="identifier">map_theme</span> </code></pre>
<a href="http://colinfay.me/wp-content/uploads/2017/05/mlp.png"><img class="aligncenter size-full wp-image-1724" src="http://colinfay.me/wp-content/uploads/2017/05/mlp.png" alt="Votes pour Marine Le Pen au second tour" width="1000" height="500" /></a>
