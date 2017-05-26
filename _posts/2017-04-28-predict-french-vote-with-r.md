---
ID: 1668
post_title: Predict french vote with R
author: colin_fay
post_date: 2017-04-28 15:08:22
post_excerpt: ""
layout: single
permalink: /predict-french-vote-with-r/
published: true
yst_is_cornerstone:
  - ""
---
## Let's try a prediction of the french presidential vote, based on the results from the first round.
<!--more-->

Before the second poll of the french election (May 7th), candidates are invited to give voting instructions to their supporters, supporting one of the candidates still in the running. Let’s try to predict the results of the second round, based on these information.

_Disclaimer: the predictions contained in this work are purely hypothetical, and rely on very strong and unrealistic assumptions, like "voters for a candidate will follow his voting instructions". _
### Creating the data
I’ve extracted the info from :
- “who supports who” on <a href="http://www.francetvinfo.fr/elections/presidentielle/fillon-melenchon-hamon-poutou-quelle-est-la-consigne-de-vote-des-neuf-elimines-en-vue-du-second-tour_2158950.html">France Tv</a>
- 1st round results on <a href="https://www.data.gouv.fr/fr/datasets/election-presidentielle-des-23-avril-et-7-mai-2017-resultats-du-1er-tour/">data.gouv</a>

Let’s create the tibble.
<pre class="r"><code class="r"><span class="keyword">library<span class="paren">(<span class="identifier">tidyverse<span class="paren">)
<span class="identifier">result <span class="operator">&lt;- <span class="identifier">tibble<span class="paren">(
  <span class="identifier">NOM <span class="operator">= <span class="identifier">c<span class="paren">(<span class="string">"MACRON",<span class="string">"MÉLENCHON",<span class="string">"FILLON",<span class="string">"LEPEN",<span class="string">"HAMON",<span class="string">"DUPONT-AIGNAN",<span class="string">"POUTOU",<span class="string">"LASSALLE",<span class="string">"ARTHAUD",<span class="string">"ASSELINEAU", <span class="string">"CHEMINADE",<span class="string">"BLANC"<span class="paren">),
  <span class="identifier">SIDE <span class="operator">= <span class="identifier">c<span class="paren">(<span class="string">"CENTRE", <span class="string">"GAUCHE", <span class="string">"DROITE",<span class="string">"DROITE",<span class="string">"GAUCHE",<span class="string">"DROITE", <span class="string">"GAUCHE", <span class="string">"SANS ETIQUETTE",<span class="string">"GAUCHE",<span class="string">"DROITE",<span class="string">"GAUCHE", <span class="string">"BLANC"<span class="paren">),
  <span class="identifier">APPEL <span class="operator">= <span class="identifier">c<span class="paren">(<span class="string">"MACRON",<span class="string">"NSP",<span class="string">"MACRON",<span class="string">"LEPEN",<span class="string">"MACRON",<span class="string">"NSP",<span class="string">"NSP",
            <span class="string">"NSP",<span class="string">"BLANC",<span class="string">"NSP",<span class="string">"NSP",<span class="string">"BLANC"<span class="paren">), 
  <span class="identifier">VOIX <span class="operator">= <span class="identifier">c<span class="paren">(<span class="number">8654331, <span class="number">7058859, <span class="number">7211121,<span class="number">7678558, <span class="number">2291025, <span class="number">1694898, <span class="number">394510, 
           <span class="number">435367, <span class="number">232439, <span class="number">332592, <span class="number">65671, <span class="number">655404<span class="paren">),
  <span class="identifier">POURC <span class="operator">= <span class="identifier">round<span class="paren">(<span class="identifier">VOIX <span class="operator">/ <span class="number">36704775 <span class="operator">* <span class="number">100, <span class="number">2<span class="paren">)
<span class="paren">)
```
Notes: in the APPEL column, the “NSP” factor means that the candidate didn’t give instructions to his/her supporters.

Let’s start by a short visualisation of the results :
<pre class="r"><code class="r"><span class="identifier">ggplot<span class="paren">(<span class="identifier">result, <span class="identifier">aes<span class="paren">(<span class="identifier">reorder<span class="paren">(<span class="identifier">NOM, <span class="identifier">POURC<span class="paren">), <span class="identifier">POURC<span class="paren">)<span class="paren">) <span class="operator">+ 
  <span class="identifier">geom_bar<span class="paren">(<span class="identifier">stat <span class="operator">= <span class="string">"identity", , <span class="identifier">fill <span class="operator">= <span class="string">"#b78d6a"<span class="paren">) <span class="operator">+ 
  <span class="identifier">coord_flip<span class="paren">(<span class="paren">) <span class="operator">+ 
  <span class="identifier">labs<span class="paren">(<span class="identifier">title <span class="operator">= <span class="string">"Résultats du premier tour en France", 
       <span class="identifier">x <span class="operator">= <span class="string">"", 
       <span class="identifier">y <span class="operator">= <span class="string">""<span class="paren">)<span class="operator">+ 
  <span class="identifier">theme_light<span class="paren">(<span class="paren">)
```
### <a href="https://colinfay.github.io/wp-content/uploads/2017/04/resultats-premier-tour.png"><img class="aligncenter size-full wp-image-1673" src="https://colinfay.github.io/wp-content/uploads/2017/04/resultats-premier-tour.png" alt="Résultats du premier tour" width="1000" height="500" /></a>
### Simulating second round results
Here are the results if everyone who voted for a candidate on the first round follows the instructions give by this candidate.
<pre class="r"><code class="r"><span class="identifier">ggplot<span class="paren">(<span class="identifier">result, <span class="identifier">aes<span class="paren">(<span class="identifier">reorder<span class="paren">(<span class="identifier">APPEL, <span class="identifier">POURC<span class="paren">), <span class="identifier">POURC<span class="paren">)<span class="paren">) <span class="operator">+ 
  <span class="identifier">geom_bar<span class="paren">(<span class="identifier">stat <span class="operator">= <span class="string">"identity", , <span class="identifier">fill <span class="operator">= <span class="string">"#b78d6a"<span class="paren">) <span class="operator">+ 
  <span class="identifier">coord_flip<span class="paren">(<span class="paren">) <span class="operator">+ 
  <span class="identifier">labs<span class="paren">(<span class="identifier">title <span class="operator">= <span class="string">"Résultats simulés du second tour en France", 
       <span class="identifier">subtitle <span class="operator">= <span class="string">"Suivi des consignes de vote",
       <span class="identifier">x <span class="operator">= <span class="string">"", 
       <span class="identifier">y <span class="operator">= <span class="string">""<span class="paren">)<span class="operator">+ 
  <span class="identifier">theme_light<span class="paren">(<span class="paren">)
```
#### <a href="https://colinfay.github.io/wp-content/uploads/2017/04/simulation-second-tour-1.png"><img class="aligncenter size-full wp-image-1674" src="https://colinfay.github.io/wp-content/uploads/2017/04/simulation-second-tour-1.png" alt="Simulation 1" width="1000" height="500" /></a>
Ok, now what do we do with the candidate who hasn't give any instruction?
#### Let’s try various scenarios.
What would happen if the NSP equally vote for each candidate?
<pre class="r"><code class="r"><span class="keyword">library<span class="paren">(<span class="identifier">stringr<span class="paren">)
<span class="identifier">sim1 <span class="operator">&lt;- <span class="identifier">result <span class="operator">%&gt;%
  <span class="identifier">group_by<span class="paren">(<span class="identifier">APPEL<span class="paren">) <span class="operator">%&gt;%
  <span class="identifier">summarise<span class="paren">(<span class="identifier">VOIX <span class="operator">= <span class="identifier">sum<span class="paren">(<span class="identifier">POURC<span class="paren">)<span class="paren">)
<span class="identifier">sim1<span class="operator">$<span class="identifier">VOIX <span class="operator">&lt;- <span class="identifier">c<span class="paren">(<span class="identifier">sim1<span class="operator">$<span class="identifier">VOIX<span class="paren">[<span class="paren">[<span class="number">1<span class="paren">]<span class="paren">], 
               <span class="identifier">sim1<span class="operator">$<span class="identifier">VOIX<span class="paren">[<span class="paren">[<span class="number">2<span class="paren">]<span class="paren">] <span class="operator">+ <span class="identifier">sim1<span class="operator">$<span class="identifier">VOIX<span class="paren">[<span class="paren">[<span class="number">4<span class="paren">]<span class="paren">]<span class="operator">/<span class="number">2,
               <span class="identifier">sim1<span class="operator">$<span class="identifier">VOIX<span class="paren">[<span class="paren">[<span class="number">3<span class="paren">]<span class="paren">]<span class="operator">+ <span class="identifier">sim1<span class="operator">$<span class="identifier">VOIX<span class="paren">[<span class="paren">[<span class="number">4<span class="paren">]<span class="paren">]<span class="operator">/<span class="number">2,
               <span class="literal">NA<span class="paren">)
<span class="identifier">sim1 <span class="operator">&lt;- <span class="identifier">na.omit<span class="paren">(<span class="identifier">sim1<span class="paren">)
<span class="identifier">ggplot<span class="paren">(<span class="identifier">sim1, <span class="identifier">aes<span class="paren">(<span class="identifier">reorder<span class="paren">(<span class="identifier">APPEL, <span class="identifier">VOIX<span class="paren">), <span class="identifier">VOIX<span class="paren">)<span class="paren">) <span class="operator">+ 
  <span class="identifier">geom_bar<span class="paren">(<span class="identifier">stat <span class="operator">= <span class="string">"identity", , <span class="identifier">fill <span class="operator">= <span class="string">"#b78d6a"<span class="paren">) <span class="operator">+ 
  <span class="identifier">coord_flip<span class="paren">(<span class="paren">) <span class="operator">+ 
  <span class="identifier">labs<span class="paren">(<span class="identifier">title <span class="operator">= <span class="string">"Résultats simulés du second tour en France",
       <span class="identifier">subtitle <span class="operator">= <span class="string">"NSP à 50 / 50 Macron - Le Pen",
       <span class="identifier">x <span class="operator">= <span class="string">"", 
       <span class="identifier">y <span class="operator">= <span class="string">""<span class="paren">)<span class="operator">+ 
  <span class="identifier">theme_light<span class="paren">(<span class="paren">)
```
<a href="https://colinfay.github.io/wp-content/uploads/2017/04/simulation-second-tour-2.png"><img class="aligncenter size-full wp-image-1678" src="https://colinfay.github.io/wp-content/uploads/2017/04/simulation-second-tour-2.png" alt="" width="1000" height="500" /></a>

Ok, we're good with that one. What would happen if all the NSP vote for Marine Le Pen?
<pre class="r"><code class="r"><span class="keyword">library<span class="paren">(<span class="identifier">stringr<span class="paren">)
<span class="identifier">result <span class="operator">%&gt;%
  <span class="identifier">mutate<span class="paren">(<span class="identifier">APPEL <span class="operator">= <span class="identifier">str_replace_all<span class="paren">(<span class="identifier">result<span class="operator">$<span class="identifier">APPEL, <span class="string">"NSP", <span class="string">"LEPEN"<span class="paren">)<span class="paren">) <span class="operator">%&gt;%
  <span class="identifier">ggplot<span class="paren">(<span class="identifier">aes<span class="paren">(<span class="identifier">reorder<span class="paren">(<span class="identifier">APPEL, <span class="identifier">POURC<span class="paren">), <span class="identifier">POURC<span class="paren">)<span class="paren">) <span class="operator">+ 
  <span class="identifier">geom_bar<span class="paren">(<span class="identifier">stat <span class="operator">= <span class="string">"identity", , <span class="identifier">fill <span class="operator">= <span class="string">"#b78d6a"<span class="paren">) <span class="operator">+ 
  <span class="identifier">coord_flip<span class="paren">(<span class="paren">) <span class="operator">+ 
  <span class="identifier">labs<span class="paren">(<span class="identifier">title <span class="operator">= <span class="string">"Résultats simulés du second tour en France", 
       <span class="identifier">subtitle <span class="operator">= <span class="string">"NSP 100% Marine Le Pen",
       <span class="identifier">x <span class="operator">= <span class="string">"", 
       <span class="identifier">y <span class="operator">= <span class="string">""<span class="paren">)<span class="operator">+ 
  <span class="identifier">theme_light<span class="paren">(<span class="paren">)
```
<a href="https://colinfay.github.io/wp-content/uploads/2017/04/simulation-second-tour-3.png"><img class="aligncenter size-full wp-image-1677" src="https://colinfay.github.io/wp-content/uploads/2017/04/simulation-second-tour-3.png" alt="" width="1000" height="500" /></a>

Aaaand that's tight, but Macron still wins. What if all NSP go to Macron?
<pre class="r"><code class="r"><span class="identifier">result <span class="operator">%&gt;%
  <span class="identifier">mutate<span class="paren">(<span class="identifier">APPEL <span class="operator">= <span class="identifier">str_replace_all<span class="paren">(<span class="identifier">result<span class="operator">$<span class="identifier">APPEL, <span class="string">"NSP", <span class="string">"MACRON"<span class="paren">)<span class="paren">) <span class="operator">%&gt;%
  <span class="identifier">ggplot<span class="paren">(<span class="identifier">aes<span class="paren">(<span class="identifier">reorder<span class="paren">(<span class="identifier">APPEL, <span class="identifier">POURC<span class="paren">), <span class="identifier">POURC<span class="paren">)<span class="paren">) <span class="operator">+ 
  <span class="identifier">geom_bar<span class="paren">(<span class="identifier">stat <span class="operator">= <span class="string">"identity", , <span class="identifier">fill <span class="operator">= <span class="string">"#b78d6a"<span class="paren">) <span class="operator">+ 
  <span class="identifier">coord_flip<span class="paren">(<span class="paren">) <span class="operator">+ 
  <span class="identifier">labs<span class="paren">(<span class="identifier">title <span class="operator">= <span class="string">"Résultats simulés du second tour en France", 
       <span class="identifier">subtitle <span class="operator">= <span class="string">"NSP 100% Macron",
       <span class="identifier">x <span class="operator">= <span class="string">"", 
       <span class="identifier">y <span class="operator">= <span class="string">""<span class="paren">)<span class="operator">+ 
  <span class="identifier">theme_light<span class="paren">(<span class="paren">)
<a href="https://colinfay.github.io/wp-content/uploads/2017/04/simulation-second-tour-4.png"><img class="aligncenter size-full wp-image-1676" src="https://colinfay.github.io/wp-content/uploads/2017/04/simulation-second-tour-4.png" alt="" width="1000" height="500" /></a>

```
Yeah, that was obvious.
### Left vs Right wing
OK, let’s try something else. What if all voters who chose a right wing candidate vote for Marine Le Pen, and voters for a left wing candidate Emmanuel Macron ?
<pre class="r"><code class="r"><span class="identifier">result <span class="operator">%&gt;%
  <span class="identifier">left_join<span class="paren">(<span class="identifier">data.frame<span class="paren">(<span class="identifier">SIDE <span class="operator">= <span class="identifier">c<span class="paren">(<span class="string">"CENTRE",<span class="string">"GAUCHE",<span class="string">"DROITE", <span class="string">"BLANC", <span class="string">"SANS ETIQUETTE"<span class="paren">), 
                       <span class="identifier">SIM <span class="operator">= <span class="identifier">c<span class="paren">(<span class="string">"MACRON",<span class="string">"MACRON",<span class="string">"LEPEN", <span class="string">"BLANC", <span class="string">"BLANC"<span class="paren">)<span class="paren">), <span class="identifier">by <span class="operator">= <span class="string">"SIDE"<span class="paren">) <span class="operator">%&gt;%
  <span class="identifier">ggplot<span class="paren">(<span class="identifier">aes<span class="paren">(<span class="identifier">reorder<span class="paren">(<span class="identifier">SIM.y, <span class="identifier">POURC<span class="paren">), <span class="identifier">POURC<span class="paren">)<span class="paren">) <span class="operator">+ 
  <span class="identifier">geom_bar<span class="paren">(<span class="identifier">stat <span class="operator">= <span class="string">"identity", , <span class="identifier">fill <span class="operator">= <span class="string">"#b78d6a"<span class="paren">) <span class="operator">+ 
  <span class="identifier">coord_flip<span class="paren">(<span class="paren">) <span class="operator">+ 
  <span class="identifier">labs<span class="paren">(<span class="identifier">title <span class="operator">= <span class="string">"Résultats simulés du second tour en France", 
       <span class="identifier">x <span class="operator">= <span class="string">"", 
       <span class="identifier">y <span class="operator">= <span class="string">""<span class="paren">)<span class="operator">+ 
  <span class="identifier">theme_light<span class="paren">(<span class="paren">)

```
<a href="https://colinfay.github.io/wp-content/uploads/2017/04/simulation-second-tour-5.png"><img class="aligncenter size-full wp-image-1675" src="https://colinfay.github.io/wp-content/uploads/2017/04/simulation-second-tour-5.png" alt="" width="1000" height="500" /></a>

Ok, still tight, but Emmanuel Macron still wins
