---
layout: post
title: Nginx SSL 结合Tomcat 重定向URL变成HTTP的问题
categories: [Nginx]
description: Nginx SSL 结合Tomcat 重定向URL变成HTTP的问题
keywords: Nginx
tags: Nginx
---

### 参考


```
http://www.siven.net/posts/d925bb5d.html#%E5%AE%9E%E8%B7%B5%E4%BA%8Ctomcat%E6%96%B0%E5%A2%9E%E9%85%8D%E7%BD%AE
```

![依赖注入方式](/images/post/201910/1.png)

## 问题描述

由于要配置服务器(Nginx + Tomcat）的SSL的问题（Nginx同时监听`HTTP`和`HTTPS`)，但是，如果用户访问的是`HTTPS`协议，然后Tomcat进行重定向的时候，却变成了`HTTP`.