---
layout: page
title: About
description: 一件意事，一个爱好，足矣
keywords: MiniPa
comments: true
menu: 关于
permalink: /about/
---

努力做一件事

亲近一个爱好

交往一些些朋友

Coding 不是万能的，但没有 Coding 是万万不能的。

## 联系

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
{% if site.url contains 'minipa' %}
<li>
微信：<br />
<img style="height:192px;width:192px;border:1px solid lightgrey;" src="{{ assets_base_url }}/assets/images/qrcode.jpg" alt="同一个世界，同一个梦想" />
</li>
{% endif %}
</ul>


## 技能关键词

{% for skill in site.data.skills %}
### {{ skill.name }}
<div class="btn-inline">
{% for keyword in skill.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
