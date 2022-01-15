---
layout: page
title: 关于
description: viakiba
keywords: viakiba
comments: false
menu: 关于
permalink: /about/
---

- 本科 河南科技大学 物联网工程专业(计算机系) 2013.9~2017.6

***
- 点点互动，Persona,后端研发，2021.11~至今
- 字节·Ohayoo,白鹭工作室，后端研发，2020.12~2021.11
- 开心网，开瑞工作室，开发工程师，2019.11~2020.12
- 雷森科技发展有限公司，研发部，Java工程师，2018.4~2019.11
- 北京龙马游戏，研发部，Java开发，2017.7~2018.10

***

- 点点互动，在研
- 字节前后参与，主公别消我，不休传说以及狗头大作战项目，其中不休传说最高DUA超50万。
- 开心网，主要负责三国群英传·争霸项目，包含 app store 以及 小程序 的后端开发。
- 雷森科技，主要负责华为，联想，ticwatch的终端设备的芯片卡虚拟化技术，例如 地铁，银行卡的手机刷NFC卡等。
- 龙马游戏，主要负责天书奇谈的页游模块开发，后期参与猫游记手游的开发。

***

{% for category in site.data.skills %}

#### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
