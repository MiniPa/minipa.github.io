---
layout: page
title: Links
description: 给我一双微暖的手，给我一条美丽的链接
keywords: 友情链接
comments: true
menu: 链接
permalink: /links/
---

> 你是一只小白兔，我也是一只小白兔，一群小白兔

<ul>
{% for link in site.data.links %}
  {% if link.src == 'life' %}
  <li><a href="{{ link.url }}" target="_blank">{{ link.name}}</a></li>
  {% endif %}
{% endfor %}
</ul>

> 友情链接

<ul>
{% for link in site.data.links %}
  {% if link.src == 'www' %}
  <li><a href="{{ link.url }}" target="_blank">{{ link.name}}</a></li>
  {% endif %}
{% endfor %}
</ul>
