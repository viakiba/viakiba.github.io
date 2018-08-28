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

{% for category in site.data.skills %}
#### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}

#### 关于 viakiba
<p>90后，17年毕业于河南科技大学物联网工程。毕业后入职于龙马游戏（前人人游戏），前期负责维护游戏[天书奇谈](https://baike.baidu.com/item/%E5%A4%A9%E4%B9%A6%E5%A5%87%E8%B0%88/6844618?fr=aladdin) 与[龙之刃](https://baike.baidu.com/item/%E9%BE%99%E4%B9%8B%E5%88%83/5276465?fr=aladdin)。主要工作是根据策划需求新增节日的游戏活动以及bug修复包括模块功能的扩展。后期负责页游我的江湖的腾讯开放平台的接入包括登陆模块以及金币系统改造。再往后参与到一即时战斗类型的手游开发。手游还未上线就不说名字了。 </p>

在此期间，技术得到了很大的历练，对于多线程，并发以及长连接有了充分的实践。使用到了技术包括

做过游戏做过基础服务，喜欢折腾。
