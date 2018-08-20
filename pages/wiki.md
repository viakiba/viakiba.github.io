---
layout: page
title: 在线工具
description: 人越学越觉得自己无知
keywords: 在线工具
comments: false
menu: 在线工具
permalink: /toollink/
---

> 改变一生只需要那么一点点时间，领悟那改变却要耗费一生。

{% for link in site.data.toollink %}
* [{{ link.name }}]({{ link.url }})
{% endfor %}
