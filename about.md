---
title: About Me
---

I am an Antenna engineer working in the Greater Seattle Area.

I specialize in:

- phased array antenna systems
- wireless communication
- digital signal processing

I like to learn about random things that may or may not be related to my field; some of them are documented on my [notes](notes.md) page.

Here are some topics I like to write about:

<ul>
  {% assign all_tags = site.tags | sort %}
  {% for tag in all_tags %}
    <li><a href="/tags/sorted/{{ tag[0] | slugify }}/">{{ tag[0] }}</a> ({{ tag[1].size }})</li>
  {% endfor %}
</ul>
