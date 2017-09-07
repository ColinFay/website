---
layout: archive
permalink: /categories/
title: "Posts by Category"
author_profile: true
---

## Jump to : 
<class = "list">
+ [R blog (en)](http://colinfay.me/categories/#r-blog-en) 
+ [R blog (fr)](http://colinfay.me/categories/#r-blog-fr)
+ [Random](http://colinfay.me/categories/#random)

___

## Subscribe to the blog feeds

+ [Full blog](http://colinfay.me/feed.xml)
+ [R blog (en)](http://colinfay.me/rblog.rss) 
+ [R blog (fr)](http://colinfay.me/rblog.rss)
+ [Random](http://colinfay.me/random.rss)
</class>
___


{% include group-by-array collection=site.posts field="categories" %}

{% for category in group_names %}
  {% assign posts = group_items[forloop.index0] %}
  <h2 id="{{ category | slugify }}" class="archive__subtitle">{{ category }}</h2>
  {% for post in posts %}
    {% include archive-single.html %}
  {% endfor %}
{% endfor %}
