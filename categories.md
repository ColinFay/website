---
layout: archive
permalink: /categories/
title: "Posts by Category"
author_profile: true
---

## Jump to : 

+ {:.list}[R blog (en)](http://colinfay.me/categories/#r-blog-en) 
+ {:.list}[R blog (fr)](http://colinfay.me/categories/#r-blog-fr)
+ {:.list}[Random](http://colinfay.me/categories/#random)

___

## Subscribe to the blog feeds

+ {:.list}[Full blog](http://colinfay.me/feed.xml)
+ {:.list}[R blog (en)](http://colinfay.me/rblog.rss) 
+ {:.list}[R blog (fr)](http://colinfay.me/rblog.rss)
+ {:.list}[Random](http://colinfay.me/random.rss)
___


{% include group-by-array collection=site.posts field="categories" %}

{% for category in group_names %}
  {% assign posts = group_items[forloop.index0] %}
  <h2 id="{{ category | slugify }}" class="archive__subtitle">{{ category }}</h2>
  {% for post in posts %}
    {% include archive-single.html %}
  {% endfor %}
{% endfor %}
