---
layout: archive
permalink: /categories/
title: "Posts by Category"
author_profile: true
---

## Jump to : 

+ [R blog (en)](http://colinfay.me/rblogen/)
+ [R blog (fr)](http://colinfay.me/rblogfr/)
+ [Random](http://colinfay.me/random)

___

{% include group-by-array collection=site.posts field="categories" %}

{% for category in group_names %}
  {% assign posts = group_items[forloop.index0] %}
  <h2 id="{{ category | slugify }}" class="archive__subtitle">{{ category }}</h2>
  {% for post in posts %}
    {% include archive-single.html %}
  {% endfor %}
{% endfor %}
