---
ID: 1137
post_title: '#RStats — languagelayeR : accéder à l&rsquo;API languagelayer avec R'
author: colin_fay
post_date: 2016-12-12 23:07:28
post_excerpt: ""
layout: post
permalink: >
  /rstats-languagelayer-api/
published: true
---
<h2>Facilitez vos fouilles de textes avec languagelayeR, un package R qui détecte la langue d'une suite de mots.<!--more--></h2>
&nbsp;

Ce package accède à l'API <a href="https://languagelayer.com/" target="_blank">languagelayer</a>, un outil de détection de la langue d'un texte.
<h3>L'API languagelayer</h3>
L'API languagelayer est une interface JSON vous permettant de détecter une langue — le package <code>languagelayeR</code> vous offre une connexion a cette interface avec R.
<h3>Installer languagelayerR</h3>
Pour installer le package depuis le CRAN :
<pre class="{r}">install.packages("languagelayeR")</pre>
Pour installer la version dev depuis <a href="https://github.com/ColinFay" target="_blank">GitHub</a>  :
<pre class="{r}">devtools::install_github("ColinFay/languagelayeR")</pre>
<h3>Comment fonctionne languagelayeR</h3>
La version 1.0.0 comprend 3 fonctions :
<ul>
 	<li><code>getLanguage</code> Effectuer une requête pour extraire la langue d'une suite de mots</li>
 	<li><code>getSupportedLanguage</code> Obtenir la liste de toutes les langues disponibles sur l'API</li>
 	<li><code>setApiKey</code> Définir la clé personnelle</li>
</ul>
<h3>Avant de commencer</h3>
La première étape de toute utilisation de ce package est la création d'un compte sur languagelayer, puis la définition de la clé API sur votre session R, via la fonction <code>setApiKey(apikey = "yourapikey")</code>.

Retrouvez votre clé API sur votre <a href="https://languagelayer.com/dashboard">dashboard</a>.
<h3>Exemples</h3>
<h4>getLanguage</h4>
Détecter une langue :
<pre class="{r}">getLanguage(query = "I really really love R and that's a good thing, right?")</pre>
<h4>getSupportedLanguage</h4>
Lister toutes les langues disponibles :
<pre class="{r}">getSupportedLanguage()</pre>
<h3>Contact</h3>
Questions et feedbacks <a href="mailto:contact@colinfay.me" target="_blank">bienvenus</a> !