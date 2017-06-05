---
layout: archive
permalink: /rblogen/
title: "Category — Random"
author_profile: true
---

{% include group-by-array collection=site.posts field="categories" %}

{% for category in group_names %} 
  {% assign posts = group_items[forloop.index0] %}
  <h2 id="{{ category | slugify }}" class="archive__subtitle">{{ category }}</h2>
  {% for post in posts %}
    {% if post.categories contains 'random' %}
      {% include archive-single.html %}
    {% endif %}
  {% endfor %}
{% endfor %}
