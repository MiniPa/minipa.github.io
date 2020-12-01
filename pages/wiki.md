---
layout: page
title: Wiki
description: Nothing 就是干
keywords: 维基, Wiki
comments: false
menu: 维基
permalink: /wiki/
---

> 来吧，快进入我的大脑吧，求您了。  
[MiniPa 的思维脑图](http://naotu.baidu.com/file/7e660fc1ceda778ca591adaba6f7cdcb?token=5f13b5f65ac59922)


<ul class="listing">
{% for wiki in site.wiki %}
{% if wiki.title != "Wiki Template" and wiki.topmost == true %}
<li class="listing-item"><a href="{{ site.url }}{{ wiki.url }}"><span class="top-most-flag">[置顶]</span>{{ wiki.title }}</a></li>
{% endif %}
{% endfor %}
{% for wiki in site.wiki %}
{% if wiki.title != "Wiki Template" and wiki.topmost != true %}
<li class="listing-item"><a href="{{ site.url }}{{ wiki.url }}">{{ wiki.title }}<span style="font-size:12px;color:red;font-style:italic;">{%if wiki.layout == 'mindmap' %}  mindmap{% endif %}</span></a></li>
{% endif %}
{% endfor %}
</ul>
