---
layout: post
title: 压测服务器端报 broken pipe
categories: [AOP, Spring]
description: 压测 tomcat 报 broken pipe
keywords: TCP, Troubleshooting
---

```
    最近协助一个项目做环境压测，前置是nginx负载到后端的服务器上。压测的时候后端的服务器报 
        org.apache.catalina.connector.ClientAbortException: java.io.IOException: Broken pipe
    
    这个问题网上解释的很多，如：
        https://my.oschina.net/u/3011256/blog/1625937
        https://blog.csdn.net/current112233/article/details/79041450
```

压测用的ngrinder
```
    https://github.com/naver/ngrinder/wiki
    写的groovy脚本 复杂的业务卸载jar里面 共groovy调用
```