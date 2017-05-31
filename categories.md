---
layout: archive
permalink: /categories/
title: "Posts by Category"
author_profile: true
---

Go to 

+ [r-blog-en](http://colinfay.me/categories/#r-blog-en)
+ [r-blog-fr](http://colinfay.me/categories/#r-blog-fr)
+ [random](http://colinfay.me/categories/#random)

{% include group-by-array collection=site.posts field="categories" %}

{% for category in group_names %}
  {% assign posts = group_items[forloop.index0] %}
  <h2 id="{{ category | slugify }}" class="archive__subtitle">{{ category }}</h2>
  {% for post in posts %}
    {% include archive-single.html %}
  {% endfor %}
{% endfor %}
