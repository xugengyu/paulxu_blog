---
layout: default
title: "Blog Posts"
permalink: /tag/blog/
---

<h1>Blog Posts</h1>

<ul>
  {% assign tag_name = "blog" %}
  {% for post in site.posts %}
    {% if post.tags contains tag_name %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endif %}
  {% endfor %}
</ul>