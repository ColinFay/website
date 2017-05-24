---
ID: 1026
post_title: 'rpinterest : un package R pour accéder à l&rsquo;API Pinterest'
author: colin_fay
post_date: 2016-07-30 20:17:45
post_excerpt: ""
layout: single
permalink: >
  /rpinterest-un-package-r-pour-acceder-a-lapi-pinterest/
published: true
---
<h2>Accéder aux boards, pins et users de Pinterest directement dans R, tel est l'object de rpinterest, un package destiné à faciliter le dialogue entre l'API du réseau social et votre logiciel favori. <!--more--></h2>
<div id="geoapi" class="section level2">
<h3></h3>
&nbsp;
<h3>Installer rpinterest</h3>
<em>Update [17/08/16]</em>

Le package est désormais disponible sur le <a href="https://cran.r-project.org/web/packages/rpinterest/index.html">CRAN</a> !
<pre class="{r}">install.packages("rpinterest")</pre>
<em>[/update]</em>

&nbsp;

Pour installer la version dev depuis <a href="https://github.com/ColinFay/rpinterest" target="_blank">GitHub</a>  :
<pre class="{r}"><code>devtools::install_github("ColinFay/rpinterest")</code></pre>
<h3>Comment fonctionne rgeoapi</h3>
La version actuelle comprend 7 fonctions :
<ul>
 	<li><code>BoardPinsByID</code> obtenir les pins d'un board à partir de l'ID d'un board</li>
 	<li><code>BoardPinsByName</code> obtenir les pins d'un board à partir du nom d'un board</li>
 	<li><code>BoardSpecByID</code> obtenir les informations sur un board à partir de l'ID d'un board</li>
 	<li><code>BoardSpecByName</code> obtenir les informations sur un board à partir du nom d'un board</li>
 	<li><code>PinSpecByID</code> obtenir les informations sur un pin à partir de son ID</li>
 	<li><code>UserSpecByID</code> obtenir les informations sur un utilisateur à partir de son ID</li>
 	<li><code>UserSpecNyName</code> obtenir les informations sur un utilisateur à partir de son nom</li>
</ul>
<h3>Obtenir un acess token</h3>
Pour utiliser ces fonctions, il est indispensable d'obtenir un <em>access token</em> disponible sur l'<a href="https://developers.pinterest.com/tools/access_token/" target="_blank">interface developpers</a> de Pinterest.

[Astuce gain de temps] Pour plus de fluidité dans votre utilisation de ce package, créez un objet R appelé <em>token</em>, et contenant la chaine de caractères de votre access token — ensuite, vous n'aurez plus qu'à insérer <em>token</em> dans votre appel à la fonction (ce qui vous sauvera de quelques mouvements de clavier, et de quelques sueurs froides, avouons-le).
<h3>Quelques examples</h3>
<h4>BoardPinsByID</h4>
Cette fonction prend l'ID d'un board et l'access token obtenu dans l'interface developpers de Pinterest, et retourne tous les pins disponibles sur ce board.
<pre class="{r}"><code><span class="pl-c">BoardPinsByID(boardID = "42080646457333782", token = token)</span></code></pre>
<h4>BoardSpecByName</h4>
Cette fonction vous permet d'obtenir les informations sur un board, à partir de son nom et du nom de l'utilisateur qui l'a créé.
<pre class="{r}"><code><span class="pl-c">BoardSpecByName(user = "colinfay", board = "blanc-mon-amour", token = token)</span></code></pre>
<h4>UserSpecByName</h4>
Comme son nom l'indique, cette fonction obtient les informations sur l’utilisateur Pinterest spécifié.
<pre class="{r}"><code><span class="pl-c">UserSpecByName(user = "colinfay", token = token)</span></code></pre>
<h3>Contact</h3>
Vos questions et feedbacks <a href="mailto:contact@colinfay.me">sont les bienvenus</a> !

</div>
