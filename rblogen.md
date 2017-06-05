---
layout: archive
permalink: /rblogen/
title: "Category — R Blog (en)"
author_profile: true
---

{% for category in group_names %} 
  {% assign posts = group_items[forloop.index0] %}
  <h2 id="{{ category | slugify }}" class="archive__subtitle">{{ category }}</h2>
  {% for post in posts %}
    {% if post.categories contains 'r-blog-en' %}
      {% include archive-single.html %}
    {% endif %}
  {% endfor %}
{% endfor %}
