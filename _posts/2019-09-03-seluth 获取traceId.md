---
layout: post
title: seluth 获取traceId
categories: [seluth]
description: seluth 获取traceId
keywords: Java, seluth
tags: Java, seluth
---

代码如下：

```java
TraceContext context = Tracing.current().tracer().currentSpan().context();
String traceIdString = context.traceIdString();
```