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
<h2>Let's try a prediction of the french presidential vote, based on the results from the first round.</h2>
<!--more-->

Before the second poll of the french election (May 7th), candidates are invited to give voting instructions to their supporters, supporting one of the candidates still in the running. Let’s try to predict the results of the second round, based on these information.

<em>Disclaimer: the predictions contained in this work are purely hypothetical, and rely on very strong and unrealistic assumptions, like "voters for a candidate will follow his voting instructions". </em>
<h3>Creating the data</h3>
I’ve extracted the info from :
- “who supports who” on <a href="http://www.francetvinfo.fr/elections/presidentielle/fillon-melenchon-hamon-poutou-quelle-est-la-consigne-de-vote-des-neuf-elimines-en-vue-du-second-tour_2158950.html">France Tv</a>
- 1st round results on <a href="https://www.data.gouv.fr/fr/datasets/election-presidentielle-des-23-avril-et-7-mai-2017-resultats-du-1er-tour/">data.gouv</a>

Let’s create the tibble.
<pre class="r"><code class="r"><span class="keyword">library</span><span class="paren">(</span><span class="identifier">tidyverse</span><span class="paren">)</span>
<span class="identifier">result</span> <span class="operator">&lt;-</span> <span class="identifier">tibble</span><span class="paren">(</span>
  <span class="identifier">NOM</span> <span class="operator">=</span> <span class="identifier">c</span><span class="paren">(</span><span class="string">"MACRON"</span>,<span class="string">"MÉLENCHON"</span>,<span class="string">"FILLON"</span>,<span class="string">"LEPEN"</span>,<span class="string">"HAMON"</span>,<span class="string">"DUPONT-AIGNAN"</span>,<span class="string">"POUTOU"</span>,<span class="string">"LASSALLE"</span>,<span class="string">"ARTHAUD"</span>,<span class="string">"ASSELINEAU"</span>, <span class="string">"CHEMINADE"</span>,<span class="string">"BLANC"</span><span class="paren">)</span>,
  <span class="identifier">SIDE</span> <span class="operator">=</span> <span class="identifier">c</span><span class="paren">(</span><span class="string">"CENTRE"</span>, <span class="string">"GAUCHE"</span>, <span class="string">"DROITE"</span>,<span class="string">"DROITE"</span>,<span class="string">"GAUCHE"</span>,<span class="string">"DROITE"</span>, <span class="string">"GAUCHE"</span>, <span class="string">"SANS ETIQUETTE"</span>,<span class="string">"GAUCHE"</span>,<span class="string">"DROITE"</span>,<span class="string">"GAUCHE"</span>, <span class="string">"BLANC"</span><span class="paren">)</span>,
  <span class="identifier">APPEL</span> <span class="operator">=</span> <span class="identifier">c</span><span class="paren">(</span><span class="string">"MACRON"</span>,<span class="string">"NSP"</span>,<span class="string">"MACRON"</span>,<span class="string">"LEPEN"</span>,<span class="string">"MACRON"</span>,<span class="string">"NSP"</span>,<span class="string">"NSP"</span>,
            <span class="string">"NSP"</span>,<span class="string">"BLANC"</span>,<span class="string">"NSP"</span>,<span class="string">"NSP"</span>,<span class="string">"BLANC"</span><span class="paren">)</span>, 
  <span class="identifier">VOIX</span> <span class="operator">=</span> <span class="identifier">c</span><span class="paren">(</span><span class="number">8654331</span>, <span class="number">7058859</span>, <span class="number">7211121</span>,<span class="number">7678558</span>, <span class="number">2291025</span>, <span class="number">1694898</span>, <span class="number">394510</span>, 
           <span class="number">435367</span>, <span class="number">232439</span>, <span class="number">332592</span>, <span class="number">65671</span>, <span class="number">655404</span><span class="paren">)</span>,
  <span class="identifier">POURC</span> <span class="operator">=</span> <span class="identifier">round</span><span class="paren">(</span><span class="identifier">VOIX</span> <span class="operator">/</span> <span class="number">36704775</span> <span class="operator">*</span> <span class="number">100</span>, <span class="number">2</span><span class="paren">)</span>
<span class="paren">)</span></code></pre>
Notes: in the APPEL column, the “NSP” factor means that the candidate didn’t give instructions to his/her supporters.

Let’s start by a short visualisation of the results :
<pre class="r"><code class="r"><span class="identifier">ggplot</span><span class="paren">(</span><span class="identifier">result</span>, <span class="identifier">aes</span><span class="paren">(</span><span class="identifier">reorder</span><span class="paren">(</span><span class="identifier">NOM</span>, <span class="identifier">POURC</span><span class="paren">)</span>, <span class="identifier">POURC</span><span class="paren">)</span><span class="paren">)</span> <span class="operator">+</span> 
  <span class="identifier">geom_bar</span><span class="paren">(</span><span class="identifier">stat</span> <span class="operator">=</span> <span class="string">"identity"</span>, , <span class="identifier">fill</span> <span class="operator">=</span> <span class="string">"#b78d6a"</span><span class="paren">)</span> <span class="operator">+</span> 
  <span class="identifier">coord_flip</span><span class="paren">(</span><span class="paren">)</span> <span class="operator">+</span> 
  <span class="identifier">labs</span><span class="paren">(</span><span class="identifier">title</span> <span class="operator">=</span> <span class="string">"Résultats du premier tour en France"</span>, 
       <span class="identifier">x</span> <span class="operator">=</span> <span class="string">""</span>, 
       <span class="identifier">y</span> <span class="operator">=</span> <span class="string">""</span><span class="paren">)</span><span class="operator">+</span> 
  <span class="identifier">theme_light</span><span class="paren">(</span><span class="paren">)</span></code></pre>
<h3><a href="https://colinfay.github.io/wp-content/uploads/2017/04/resultats-premier-tour.png"><img class="aligncenter size-full wp-image-1673" src="https://colinfay.github.io/wp-content/uploads/2017/04/resultats-premier-tour.png" alt="Résultats du premier tour" width="1000" height="500" /></a></h3>
<h3>Simulating second round results</h3>
Here are the results if everyone who voted for a candidate on the first round follows the instructions give by this candidate.
<pre class="r"><code class="r"><span class="identifier">ggplot</span><span class="paren">(</span><span class="identifier">result</span>, <span class="identifier">aes</span><span class="paren">(</span><span class="identifier">reorder</span><span class="paren">(</span><span class="identifier">APPEL</span>, <span class="identifier">POURC</span><span class="paren">)</span>, <span class="identifier">POURC</span><span class="paren">)</span><span class="paren">)</span> <span class="operator">+</span> 
  <span class="identifier">geom_bar</span><span class="paren">(</span><span class="identifier">stat</span> <span class="operator">=</span> <span class="string">"identity"</span>, , <span class="identifier">fill</span> <span class="operator">=</span> <span class="string">"#b78d6a"</span><span class="paren">)</span> <span class="operator">+</span> 
  <span class="identifier">coord_flip</span><span class="paren">(</span><span class="paren">)</span> <span class="operator">+</span> 
  <span class="identifier">labs</span><span class="paren">(</span><span class="identifier">title</span> <span class="operator">=</span> <span class="string">"Résultats simulés du second tour en France"</span>, 
       <span class="identifier">subtitle</span> <span class="operator">=</span> <span class="string">"Suivi des consignes de vote"</span>,
       <span class="identifier">x</span> <span class="operator">=</span> <span class="string">""</span>, 
       <span class="identifier">y</span> <span class="operator">=</span> <span class="string">""</span><span class="paren">)</span><span class="operator">+</span> 
  <span class="identifier">theme_light</span><span class="paren">(</span><span class="paren">)</span></code></pre>
<h4><a href="https://colinfay.github.io/wp-content/uploads/2017/04/simulation-second-tour-1.png"><img class="aligncenter size-full wp-image-1674" src="https://colinfay.github.io/wp-content/uploads/2017/04/simulation-second-tour-1.png" alt="Simulation 1" width="1000" height="500" /></a></h4>
Ok, now what do we do with the candidate who hasn't give any instruction?
<h4>Let’s try various scenarios.</h4>
What would happen if the NSP equally vote for each candidate?
<pre class="r"><code class="r"><span class="keyword">library</span><span class="paren">(</span><span class="identifier">stringr</span><span class="paren">)</span>
<span class="identifier">sim1</span> <span class="operator">&lt;-</span> <span class="identifier">result</span> <span class="operator">%&gt;%</span>
  <span class="identifier">group_by</span><span class="paren">(</span><span class="identifier">APPEL</span><span class="paren">)</span> <span class="operator">%&gt;%</span>
  <span class="identifier">summarise</span><span class="paren">(</span><span class="identifier">VOIX</span> <span class="operator">=</span> <span class="identifier">sum</span><span class="paren">(</span><span class="identifier">POURC</span><span class="paren">)</span><span class="paren">)</span>
<span class="identifier">sim1</span><span class="operator">$</span><span class="identifier">VOIX</span> <span class="operator">&lt;-</span> <span class="identifier">c</span><span class="paren">(</span><span class="identifier">sim1</span><span class="operator">$</span><span class="identifier">VOIX</span><span class="paren">[</span><span class="paren">[</span><span class="number">1</span><span class="paren">]</span><span class="paren">]</span>, 
               <span class="identifier">sim1</span><span class="operator">$</span><span class="identifier">VOIX</span><span class="paren">[</span><span class="paren">[</span><span class="number">2</span><span class="paren">]</span><span class="paren">]</span> <span class="operator">+</span> <span class="identifier">sim1</span><span class="operator">$</span><span class="identifier">VOIX</span><span class="paren">[</span><span class="paren">[</span><span class="number">4</span><span class="paren">]</span><span class="paren">]</span><span class="operator">/</span><span class="number">2</span>,
               <span class="identifier">sim1</span><span class="operator">$</span><span class="identifier">VOIX</span><span class="paren">[</span><span class="paren">[</span><span class="number">3</span><span class="paren">]</span><span class="paren">]</span><span class="operator">+</span> <span class="identifier">sim1</span><span class="operator">$</span><span class="identifier">VOIX</span><span class="paren">[</span><span class="paren">[</span><span class="number">4</span><span class="paren">]</span><span class="paren">]</span><span class="operator">/</span><span class="number">2</span>,
               <span class="literal">NA</span><span class="paren">)</span>
<span class="identifier">sim1</span> <span class="operator">&lt;-</span> <span class="identifier">na.omit</span><span class="paren">(</span><span class="identifier">sim1</span><span class="paren">)</span>
<span class="identifier">ggplot</span><span class="paren">(</span><span class="identifier">sim1</span>, <span class="identifier">aes</span><span class="paren">(</span><span class="identifier">reorder</span><span class="paren">(</span><span class="identifier">APPEL</span>, <span class="identifier">VOIX</span><span class="paren">)</span>, <span class="identifier">VOIX</span><span class="paren">)</span><span class="paren">)</span> <span class="operator">+</span> 
  <span class="identifier">geom_bar</span><span class="paren">(</span><span class="identifier">stat</span> <span class="operator">=</span> <span class="string">"identity"</span>, , <span class="identifier">fill</span> <span class="operator">=</span> <span class="string">"#b78d6a"</span><span class="paren">)</span> <span class="operator">+</span> 
  <span class="identifier">coord_flip</span><span class="paren">(</span><span class="paren">)</span> <span class="operator">+</span> 
  <span class="identifier">labs</span><span class="paren">(</span><span class="identifier">title</span> <span class="operator">=</span> <span class="string">"Résultats simulés du second tour en France"</span>,
       <span class="identifier">subtitle</span> <span class="operator">=</span> <span class="string">"NSP à 50 / 50 Macron - Le Pen"</span>,
       <span class="identifier">x</span> <span class="operator">=</span> <span class="string">""</span>, 
       <span class="identifier">y</span> <span class="operator">=</span> <span class="string">""</span><span class="paren">)</span><span class="operator">+</span> 
  <span class="identifier">theme_light</span><span class="paren">(</span><span class="paren">)</span></code></pre>
<a href="https://colinfay.github.io/wp-content/uploads/2017/04/simulation-second-tour-2.png"><img class="aligncenter size-full wp-image-1678" src="https://colinfay.github.io/wp-content/uploads/2017/04/simulation-second-tour-2.png" alt="" width="1000" height="500" /></a>

Ok, we're good with that one. What would happen if all the NSP vote for Marine Le Pen?
<pre class="r"><code class="r"><span class="keyword">library</span><span class="paren">(</span><span class="identifier">stringr</span><span class="paren">)</span>
<span class="identifier">result</span> <span class="operator">%&gt;%</span>
  <span class="identifier">mutate</span><span class="paren">(</span><span class="identifier">APPEL</span> <span class="operator">=</span> <span class="identifier">str_replace_all</span><span class="paren">(</span><span class="identifier">result</span><span class="operator">$</span><span class="identifier">APPEL</span>, <span class="string">"NSP"</span>, <span class="string">"LEPEN"</span><span class="paren">)</span><span class="paren">)</span> <span class="operator">%&gt;%</span>
  <span class="identifier">ggplot</span><span class="paren">(</span><span class="identifier">aes</span><span class="paren">(</span><span class="identifier">reorder</span><span class="paren">(</span><span class="identifier">APPEL</span>, <span class="identifier">POURC</span><span class="paren">)</span>, <span class="identifier">POURC</span><span class="paren">)</span><span class="paren">)</span> <span class="operator">+</span> 
  <span class="identifier">geom_bar</span><span class="paren">(</span><span class="identifier">stat</span> <span class="operator">=</span> <span class="string">"identity"</span>, , <span class="identifier">fill</span> <span class="operator">=</span> <span class="string">"#b78d6a"</span><span class="paren">)</span> <span class="operator">+</span> 
  <span class="identifier">coord_flip</span><span class="paren">(</span><span class="paren">)</span> <span class="operator">+</span> 
  <span class="identifier">labs</span><span class="paren">(</span><span class="identifier">title</span> <span class="operator">=</span> <span class="string">"Résultats simulés du second tour en France"</span>, 
       <span class="identifier">subtitle</span> <span class="operator">=</span> <span class="string">"NSP 100% Marine Le Pen"</span>,
       <span class="identifier">x</span> <span class="operator">=</span> <span class="string">""</span>, 
       <span class="identifier">y</span> <span class="operator">=</span> <span class="string">""</span><span class="paren">)</span><span class="operator">+</span> 
  <span class="identifier">theme_light</span><span class="paren">(</span><span class="paren">)</span></code></pre>
<a href="https://colinfay.github.io/wp-content/uploads/2017/04/simulation-second-tour-3.png"><img class="aligncenter size-full wp-image-1677" src="https://colinfay.github.io/wp-content/uploads/2017/04/simulation-second-tour-3.png" alt="" width="1000" height="500" /></a>

Aaaand that's tight, but Macron still wins. What if all NSP go to Macron?
<pre class="r"><code class="r"><span class="identifier">result</span> <span class="operator">%&gt;%</span>
  <span class="identifier">mutate</span><span class="paren">(</span><span class="identifier">APPEL</span> <span class="operator">=</span> <span class="identifier">str_replace_all</span><span class="paren">(</span><span class="identifier">result</span><span class="operator">$</span><span class="identifier">APPEL</span>, <span class="string">"NSP"</span>, <span class="string">"MACRON"</span><span class="paren">)</span><span class="paren">)</span> <span class="operator">%&gt;%</span>
  <span class="identifier">ggplot</span><span class="paren">(</span><span class="identifier">aes</span><span class="paren">(</span><span class="identifier">reorder</span><span class="paren">(</span><span class="identifier">APPEL</span>, <span class="identifier">POURC</span><span class="paren">)</span>, <span class="identifier">POURC</span><span class="paren">)</span><span class="paren">)</span> <span class="operator">+</span> 
  <span class="identifier">geom_bar</span><span class="paren">(</span><span class="identifier">stat</span> <span class="operator">=</span> <span class="string">"identity"</span>, , <span class="identifier">fill</span> <span class="operator">=</span> <span class="string">"#b78d6a"</span><span class="paren">)</span> <span class="operator">+</span> 
  <span class="identifier">coord_flip</span><span class="paren">(</span><span class="paren">)</span> <span class="operator">+</span> 
  <span class="identifier">labs</span><span class="paren">(</span><span class="identifier">title</span> <span class="operator">=</span> <span class="string">"Résultats simulés du second tour en France"</span>, 
       <span class="identifier">subtitle</span> <span class="operator">=</span> <span class="string">"NSP 100% Macron"</span>,
       <span class="identifier">x</span> <span class="operator">=</span> <span class="string">""</span>, 
       <span class="identifier">y</span> <span class="operator">=</span> <span class="string">""</span><span class="paren">)</span><span class="operator">+</span> 
  <span class="identifier">theme_light</span><span class="paren">(</span><span class="paren">)
<a href="https://colinfay.github.io/wp-content/uploads/2017/04/simulation-second-tour-4.png"><img class="aligncenter size-full wp-image-1676" src="https://colinfay.github.io/wp-content/uploads/2017/04/simulation-second-tour-4.png" alt="" width="1000" height="500" /></a>
</span></code></pre>
Yeah, that was obvious.
<h3>Left vs Right wing</h3>
OK, let’s try something else. What if all voters who chose a right wing candidate vote for Marine Le Pen, and voters for a left wing candidate Emmanuel Macron ?
<pre class="r"><code class="r"><span class="identifier">result</span> <span class="operator">%&gt;%</span>
  <span class="identifier">left_join</span><span class="paren">(</span><span class="identifier">data.frame</span><span class="paren">(</span><span class="identifier">SIDE</span> <span class="operator">=</span> <span class="identifier">c</span><span class="paren">(</span><span class="string">"CENTRE"</span>,<span class="string">"GAUCHE"</span>,<span class="string">"DROITE"</span>, <span class="string">"BLANC"</span>, <span class="string">"SANS ETIQUETTE"</span><span class="paren">)</span>, 
                       <span class="identifier">SIM</span> <span class="operator">=</span> <span class="identifier">c</span><span class="paren">(</span><span class="string">"MACRON"</span>,<span class="string">"MACRON"</span>,<span class="string">"LEPEN"</span>, <span class="string">"BLANC"</span>, <span class="string">"BLANC"</span><span class="paren">)</span><span class="paren">)</span>, <span class="identifier">by</span> <span class="operator">=</span> <span class="string">"SIDE"</span><span class="paren">)</span> <span class="operator">%&gt;%</span>
  <span class="identifier">ggplot</span><span class="paren">(</span><span class="identifier">aes</span><span class="paren">(</span><span class="identifier">reorder</span><span class="paren">(</span><span class="identifier">SIM.y</span>, <span class="identifier">POURC</span><span class="paren">)</span>, <span class="identifier">POURC</span><span class="paren">)</span><span class="paren">)</span> <span class="operator">+</span> 
  <span class="identifier">geom_bar</span><span class="paren">(</span><span class="identifier">stat</span> <span class="operator">=</span> <span class="string">"identity"</span>, , <span class="identifier">fill</span> <span class="operator">=</span> <span class="string">"#b78d6a"</span><span class="paren">)</span> <span class="operator">+</span> 
  <span class="identifier">coord_flip</span><span class="paren">(</span><span class="paren">)</span> <span class="operator">+</span> 
  <span class="identifier">labs</span><span class="paren">(</span><span class="identifier">title</span> <span class="operator">=</span> <span class="string">"Résultats simulés du second tour en France"</span>, 
       <span class="identifier">x</span> <span class="operator">=</span> <span class="string">""</span>, 
       <span class="identifier">y</span> <span class="operator">=</span> <span class="string">""</span><span class="paren">)</span><span class="operator">+</span> 
  <span class="identifier">theme_light</span><span class="paren">(</span><span class="paren">)
</span></code></pre>
<a href="https://colinfay.github.io/wp-content/uploads/2017/04/simulation-second-tour-5.png"><img class="aligncenter size-full wp-image-1675" src="https://colinfay.github.io/wp-content/uploads/2017/04/simulation-second-tour-5.png" alt="" width="1000" height="500" /></a>

Ok, still tight, but Emmanuel Macron still wins
