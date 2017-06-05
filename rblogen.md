---
layout: archive
permalink: /rblogen/
title: "Category — R Blog (en)"
author_profile: true
---

{% for post in site.posts %}
{% if post.categories contains 'r-blog-en' %}
<entry>
        <title>{{ post.title }}</title>
        <link href="{{ site.production_url }}{{ post.url }}"/>
        <updated>{{ post.date | date_to_xmlschema }}</updated>
        <id>{{ site.production_url }}{{ post.id }}</id>
        <content type="html">{{ post.content | xml_escape }}</content>
    </entry>
{% endif %}
{% endfor %}