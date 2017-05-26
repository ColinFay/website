---
ID: 1264
post_title: >
  rgeoapi — A package to access the
  GéoAPI
author: colin_fay
post_date: 2016-06-05 19:11:29
post_excerpt: ""
layout: single
permalink: /rgeoapi-package-r/
published: true
---
<div id="destine-a-interroger-la-geoapi-detalab.-lobjectif-simplifier-lacces-a-la-reference-geographique-des-communes-francaises." class="section level2">
## rgeoapi connects R to the GéoAPI, in order to get information about french geography.
<!--more-->
rgeoapi is now on <a href="https://cran.r-project.org/web/packages/rgeoapi/">CRAN</a>
# <a id="user-content-rgeoapi" class="anchor" href="https://github.com/ColinFay/rgeoapi#rgeoapi"></a>rgeoapi
This package requests informations from the French GeoAPI inside R — <a href="https://api.gouv.fr/explorer/geoapi/">https://api.gouv.fr/explorer/geoapi/</a>
## <a id="user-content-geoapi" class="anchor" href="https://github.com/ColinFay/rgeoapi#geoapi"></a>GeoAPI
Developped by Etalab, with La Poste, l’INSEE and OpenStreetMap, the <a href="https://api.gouv.fr/explorer/geoapi/">GeoAPI</a> API is a JSON interface designed to make requests on the French geographic database.

rgeoapi was developped to facilitate your geographic projects by giving you access to these informations straight inside R. With <code>rgeoapi</code>, you can get any coordinate, size and population of a French city, to be used in your maps.

For an optimal compatibility, all the names (especially outputs) used in this package are the same as the ones used in the GeoAPI. Please note that this package works only with French cities.
## <a id="user-content-install-rgeoapi" class="anchor" href="https://github.com/ColinFay/rgeoapi#install-rgeoapi"></a>Install rgeoapi
Install this package directly in R :
<div class="highlight highlight-source-r">
<pre><span class="pl-e">devtools<span class="pl-k">::install_github(<span class="pl-s"><span class="pl-pds">"ColinFay/rgeoapi<span class="pl-pds">")</pre>
</div>
## How rgeoapi works
The version 1.0.0 works with eleven functions. Which are :
<ul>
 	<li><code>ComByCode</code> Get City by INSEE Code</li>
 	<li><code>ComByCoord</code> Get City by Coordinates</li>
 	<li><code>ComByDep</code> Get Cities by Department</li>
 	<li><code>ComByName</code> Get City by Name</li>
 	<li><code>ComByPostal</code> Get City by Postal Code</li>
 	<li><code>ComByReg</code> Get Cities by Region</li>
 	<li><code>DepByCode</code> Get Department by INSEE Code</li>
 	<li><code>DepByName</code> Get Department by Name</li>
 	<li><code>DepByReg</code> Get Departments by Region</li>
 	<li><code>RegByCode</code> Get Region by INSEE Code</li>
 	<li><code>RegByName</code> Get Region by Name</li>
</ul>
## How the functions are constructed
In the <a href="https://api.gouv.fr/explorer/geoapi/">GeoAPI</a>, you can request for "Commune", "Département" or "Région". All the functions are constructed using this terminology : AByB.
<ul>
 	<li>A being the output you need -- Com for "Commune" (refering to French cities), Dep for Département (for Department) and Reg for Région.</li>
 	<li>B being the request parameter -- Code for INSEE Code, Coord for Coordinates (WGS-84), Dep for Department, Name for name, Postal for Postal Code and Reg for Region.</li>
</ul>
## Some examples
### ComByCoord
Takes the latitude and longitude of a city, returns a data.frame with name, INSEE code, postal code, INSEE department code, INSEE region code, population (approx), surface (in hectares), lat and long (WGS-84).
<div class="highlight highlight-source-r">
<pre>ComByCoord(<span class="pl-v">lat <span class="pl-k">= <span class="pl-s"><span class="pl-pds">"48.11023<span class="pl-pds">", <span class="pl-v">lon <span class="pl-k">= <span class="pl-s"><span class="pl-pds">"-1.678872<span class="pl-pds">")</pre>
</div>
### DepByName
This function takes a character string with the name of the department, and returns a data.frame with name, INSEE code, and region code. Partial matches are possible. In that case, pertinence scores are given.
<div class="highlight highlight-source-r">
<pre>DepByName(<span class="pl-s"><span class="pl-pds">"morbihan<span class="pl-pds">")
DepByName(<span class="pl-s"><span class="pl-pds">"Il<span class="pl-pds">")</pre>
</div>
### RegByCode
This function takes an INSEE Code, returns a data.frame with name and region code.
<div class="highlight highlight-source-r">
<pre>RegByCode(<span class="pl-c1">53)</pre>
</div>
### French Tutorial &amp; contact
A French tutorial on <a href="http://colinfay.me/rgeoapi-v1/">my website</a>. Questions and feedbacks <a href="mailto:contact@colinfay.me">welcome</a> !

</div>
