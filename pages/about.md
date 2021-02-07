---
layout: page
title: About
description: 优秀的软件工程师
keywords: Yawei Wang, PfCStyle, Will, Zhuang Ma, 王亚威
comments: true
menu: 关于
permalink: /about/
---

王亚威

思考 + 前进

## 联系

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
{% if site.url contains 'pfcstyle.github.io' %}
<!-- <li>
微信公众号：<br />
<img style="height:192px;width:192px;border:1px solid lightgrey;" src="{{ assets_base_url }}/assets/images/qrcode.jpg" alt="闷骚的程序员" />
</li>
{% endif %} -->
</ul>


## Skill keywords

{% for skill in site.data.skills %}
### {{ skill.name }}
<div class="btn-inline">
{% for keyword in skill.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
