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

<!-- {% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %} -->

{% for category in site.data.skills %}

#### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}


- 本科 河南科技大学 物联网工程专业(计算机系) 2013.9~2017.6

***

- 开心网，开瑞工作室，开发工程师，2019.11~至今
- 雷森科技发展有限公司，研发部，Java工程师，2018.4~2019.11
- 北京龙马游戏，研发部，Java开发，2017.7~2018.10

***

- 主要使用 Java 进行开发工作，偶尔也会学习使用 Python 写工具解决一些重复性的问题。
- 常用 Guava 工具库来提高开发效率，也会使用 Lamada表达式使代码更简洁。
- 在游戏开发时期网络IO使用 Mina框架，负责新需求的开发，比如 开挂消息限频，活动，礼物开发。
- 常使用数据库 MySQL, Redis。
- 使用 Spring, Eureka, Feign, Ribbon, Zuul, Hystrix, MyBatis 等框架，完成企业级的系统实现并商用。
- 了解 Html, CSS, JavaScript, Jquery, EasyUI 等前端技术。
- 熟悉 GIT, SVN, IDEA, Eclipse, Nginx, Jenkins, Bash 等工具。
- 了解 Docker 容器技术。

***
