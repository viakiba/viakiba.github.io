---
layout: page
title: 关于
description: viakiba
keywords: viakiba
comments: false
menu: 关于
permalink: /about/
---

改变一生只需要那么一点点时间，领悟那改变却要耗费一生。

#### 门户

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

#### Skill Keywords

{% for category in site.data.skills %}
#### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}

> 关于 viakiba
90后，喜欢折腾。做过游戏做过基础服务，喜欢折腾。
