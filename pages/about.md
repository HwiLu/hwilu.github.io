---
layout: page
title: About
description: 天天向上，日日自省。
keywords: HwiLu
comments: true
menu: 关于
permalink: /about/
---
Day Day Up.

坚持输出，期待早日成为大牛。

## 联系

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}

## 欢迎关注
<div style="text-align:left">
    <img style="border:1px solid #dddddd;height:168px;width:168px" src="{{ site.url }}/assets/images/qrcode.jpg" alt="Hadoop集群运维" />
</div>

