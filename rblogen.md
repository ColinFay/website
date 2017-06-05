---
layout: archive
permalink: /rblogen/
title: "Category — R Blog (en)"
author_profile: true
---

{% for category in group_names %}
{% if category contains 'r-blog-en' %}
  {% assign posts = group_items[forloop.index0] %}
  <h2 id="{{ category | slugify }}" class="archive__subtitle">{{ category }}</h2>
  {% for post in posts %}
    {% include archive-single.html %}
  {% endfor %}
  {% endif %}
{% endfor %}
