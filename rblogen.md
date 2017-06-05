---
layout: archive
permalink: /rblogen/
title: "Category — R Blog (en)"
author_profile: true
---

{% include group-by-array collection=site.posts field="categories" %}

{% for category in group_names %} 
{% if category contains 'r-blog-en' %}
  {% assign posts = group_items[forloop.index0] %}
  <h2 id="{{ category | slugify }}" class="archive__subtitle">{{ category }}</h2>
  {% for post in posts %}
  {% if post.categories contains 'r-blog-en' %}
    {% include archive-single.html %}
  {% endfor %}
{% endif %}
{% endfor %}
