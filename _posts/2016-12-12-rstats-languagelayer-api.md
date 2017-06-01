---
ID: 1137
title: "#RStats — languagelayeR : accéder à l'API languagelayer avec R"
author: colin_fay
post_date: 2016-12-12 23:07:28
post_excerpt: ""
layout: single
permalink: /rstats-languagelayer-api/
published: true
categories : r-blog-fr
---
## Facilitez vos fouilles de textes avec languagelayeR, un package R qui détecte la langue d'une suite de mots.<!--more-->
&nbsp;

Ce package accède à l'API <a href="https://languagelayer.com/" target="_blank">languagelayer</a>, un outil de détection de la langue d'un texte.
### L'API languagelayer
L'API languagelayer est une interface JSON vous permettant de détecter une langue — le package `languagelayeR` vous offre une connexion a cette interface avec R.

### Installer languagelayerR
Pour installer le package depuis le CRAN :
{% highlight r %}
install.packages("languagelayeR")
{% endhighlight %}

Pour installer la version dev depuis <a href="https://github.com/ColinFay" target="_blank">GitHub</a>  :
{% highlight r %}
devtools::install_github("ColinFay/languagelayeR")
{% endhighlight %}

### Comment fonctionne languagelayeR
La version 1.0.0 comprend 3 fonctions :
{% highlight r %} 
getLanguage()
{% endhighlight %} 
Effectuer une requête pour extraire la langue d'une suite de mots
{% highlight r %} 
getSupportedLanguage
{% endhighlight %} 
Obtenir la liste de toutes les langues disponibles sur l'API</li>

{% highlight r %} 
setApiKey()
{% endhighlight %} 
Définir la clé personnelle

### Avant de commencer
La première étape de toute utilisation de ce package est la création d'un compte sur languagelayer, puis la définition de la clé API sur votre session R, via la fonction 
{% highlight r %} 
setApiKey(apikey = "yourapikey")
{% endhighlight %}

Retrouvez votre clé API sur votre <a href="https://languagelayer.com/dashboard">dashboard</a>.

### Exemples
#### getLanguage
Détecter une langue :
{% highlight r %}
getLanguage(query = "I really really love R and that's a good thing, right?")
{% endhighlight %}
#### getSupportedLanguage
Lister toutes les langues disponibles :
{% highlight r %}getSupportedLanguage(){% endhighlight %}

### Contact
Questions et feedbacks <a href="mailto:contact@colinfay.me" target="_blank">bienvenus</a> !
