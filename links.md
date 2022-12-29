---
layout: default
title: Links
---
<h1>Latest Links</h1>

<ul>
  {% for post in site.categories.link %}
    <li>
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>