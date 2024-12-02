---
title: All Tags
permalink: /tags/
---

<h2>All Tags</h2>
<ul>
{% for tag in site.tags %}
  <li><a href="{{ site.baseurl }}/tags/{{ tag[0] }}">{{ tag[0] }}</a></li>
{% endfor %}
</ul>