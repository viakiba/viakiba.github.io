---
layout: page
title: 链接
description: 没有链接的博客是孤独的
keywords: 友情链接
comments: false
menu: 链接
permalink: /toollink/
---

> 改变一生只需要那么一点点时间，领悟那改变却要耗费一生。

### 友链

{% for link in site.data.friendlink %}
* [{{ link.name }}]({{ link.url }})
{% endfor %}

### 推荐技术博主

{% for link in site.data.links %}
* [{{ link.name }}]({{ link.url }})
{% endfor %}

### 常用工具平台

{% for link in site.data.apiservice %}
* [{{ link.name }}]({{ link.url }})
{% endfor %}

### 常用在线工具

{% for link in site.data.toollink %}
* [{{ link.name }}]({{ link.url }})
{% endfor %}
