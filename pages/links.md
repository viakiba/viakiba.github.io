---
layout: page
title: 链接
description: 没有链接的博客是孤独的
keywords: 友情链接
comments: true
menu: 链接
permalink: /links/
---

> 改变一生只需要那么一点点时间，领悟那改变却要耗费一生。

{% for link in site.data.links %}
* [{{ link.name }}]({{ link.url }})
{% endfor %}
