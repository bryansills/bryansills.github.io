---
layout: default
title: Links
---
<h1>Bryan's Latest Links</h1>

{% for post in site.categories.link %}
  <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
  <time>{{ post.date | date: "%A %B %e, %Y" }}</time>
  {{ post.content }}
{% endfor %}
