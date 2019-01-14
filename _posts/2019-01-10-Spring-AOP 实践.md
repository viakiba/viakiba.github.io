---
layout: post
title: Spring-AOP 实践
categories: [AOP, Spring]
description: Spring-AOP 实践
keywords: Spring, AOP
---

### 引入
最近重构公司项目刷新缓存操作，我们的配置信息在系统的启动的时候都会执行init操作。把配置信息从数据库中加载到redis缓存中，管理系统更改配置后，需要通知到响应的模块重新执行init操作，把redis的信息刷新。要把原来的重启操作，替换成上述的这种。本着修改文件尽可能少的原则，我在响应的模块添加了一个 HTTP 接口，用于回调（接口安全按下不表），在管理系统执行了修改等操作后，在操作后进行AOP拦截，发起对应配置信息的回调刷新。由于我们的管理系统更新等操作，返回给页面的是同样的json 的 key-value 。所以我就在controoler返回值那里进行拦截处理，对该方法加上自定义的注解及注解的属性。根据属性确定刷新配置的类型。

参考
```
注解：
  https://blog.csdn.net/qq_27093465/article/details/52664401
AOP：
  https://jinnianshilongnian.iteye.com/blog/1415606
trouble-shoting:
  https://www.cnblogs.com/rickzhai/p/6504355.html
```

### DEMO

```
https://github.com/viakiba/viakiba/tree/master/Spring-aop-demo
```
核心方法：
```
@Aspect
@Component
public class SeiCallUpCache {
//解释见 https://jinnianshilongnian.iteye.com/blog/1415606
    @AfterReturning(value = "within(com.example.demo.controller..*) && @annotation(com.example.demo.annotation.UpCacheCallTsmAnnotation)" , returning = "str")
    public void upCache(JoinPoint joinPoint, Object str) throws Exception {
        UpCacheCallTsmAnnotation upCacheCallTsmAnnotation = ((MethodSignature)joinPoint.getSignature()).getMethod().getAnnotation(UpCacheCallTsmAnnotation.class);
        String type = systemLog.type();
        System.out.println("类型"+type+"str"+str);
    }
}
```

### 问题
在我那个项目里面，要把AOP拦截的如下配置放在Spring-mvc配置文件中。
```
<aop:aspectj-autoproxy/>
```
原因：
Spring和SpringMVC是有父子容器关系的，spring是一个大的父容器，springmvc是其中的一个子容器。父容器不能访问子容器对象，但是子容器可以访问父容器对象。）一般做一个ssm框架项目的时候，扫描@controller注解类的对象是在springmvc容器中。而扫描@service、@component、@Repository等注解类的对象都是在spring容器中。所以要把上面的配置放在Spring-mvc中。
