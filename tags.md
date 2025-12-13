---
layout: default
title: "Tags"
permalink: /tags/
---

<h1>All Tags</h1>

<ul>
  {% assign all_tags = site.tags | sort %}
  {% for tag in all_tags %}
    <li><a href="/tags/sorted/{{ tag[0] | slugify }}/">{{ tag[0] }}</a> ({{ tag[1].size }})</li>
  {% endfor %}
</ul>