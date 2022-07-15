---
layout: post
title: 使用lucy-xss-servlet-filter实现WEB-XSS防御
categories: [lucy-xss-servlet-filter, XSS]
description: 使用lucy-xss实现WEB-XSS防御
keywords: lucy-xss, XSS
tags: lucy-xss, XSS
---
##  概述
### XSS
[XSS](https://tech.meituan.com/fe_security.html)是一种网站应用程序的安全漏洞攻击，是代码注入的一种。它允许恶意用户将代码注入到网页上，其他用户在观看网页时就会受到影响。这类攻击通常包含了HTML以及用户端脚本语言。
### lucy-xss-servlet-filter
lucy-xss-servlet-filter 是一个基于 web 的 filter 实现的一个XSS防御方案，他的 XssFilter 模式可以支持 HTML 标记的接受 （ 例如 邮件，访客留言簿，留言板服务）。 并且支出过滤规则的自定义。XssPreventer 是基于 apache-common-lang3 实现的XSS攻击防御，这个比较简单。
### 引入

```
<!-- https://mvnrepository.com/artifact/com.navercorp.lucy/lucy-xss-servlet -->
<dependency>
    <groupId>com.navercorp.lucy</groupId>
    <artifactId>lucy-xss-servlet</artifactId>
    <version>2.0.0</version>
</dependency>
```

## web.xml 配置

```
<filter>
	<filter-name>xssEscapeServletFilter</filter-name>
	<filter-class>com.navercorp.lucy.security.xss.servletfilter.XssEscapeServletFilter</filter-class>
</filter>
<!--  这里可以多个url 可以自己定 -->
<filter-mapping>
    <filter-name>xssEscapeServletFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```
也可以使用注解方案 集成 XssEscapeServletFilter 类，全部实现他的方法，内部全用 supper调用父类即可，WebFilter 注解。
## 定义规则
在可以输出到 classpath 的文件夹下定义 lucy-xss-servlet-filter-rule.xml 文件里的规则，并指定防御模式。例如maven 工程结构下，放在resource下。
有关详细用法，请参阅以下内容

[manual.md](https://github.com/naver/lucy-xss-servlet-filter/blob/master/doc/manual.md)

[lucy-xss-servlet-filter-rule](https://github.com/naver/lucy-xss-servlet-filter/blob/master/src/test/resources/lucy-xss-servlet-filter-rule.xml)

### 规则解释样例
```
<?xml version="1.0" encoding="UTF-8"?>
<config xmlns="http://www.navercorp.com/lucy-xss-servlet">
   <defenders>

      <!-- 两种防御实现 -->
      
      <!--  注册防御器 -->
       <defender>
           <name>xssPreventerDefender</name>
           <class>com.navercorp.lucy.security.xss.servletfilter.defender.XssPreventerDefender</class>
       </defender>
       <!--
       <defender>
           <name>xssFilterDefender</name>
           <class>com.navercorp.lucy.security.xss.servletfilter.defender.XssFilterDefender</class>
           <init-param>
               <param-value>lucy-xss.xml</param-value>    
               <param-value>false</param-value>         
           </init-param>
       </defender>
       -->
   </defenders>
    <!-- default defender -->
    <default>
        <defender>xssPreventerDefender</defender>
    </default>
    <!-- global setting -->
    <global>
        <params>
        <!-- 全局 提交的参数，content的值不做处理 -->
            <param name="options" useDefender="false" />
        </params>
    </global>
    <!-- url rule setting -->
    <url-rule-set>
        <url-rule>
            <url>/script/api/validate</url>
            <params>
            <!-- 对于上述url,提交的参数，content的值不做处理  -->
                <param name="content" useDefender="false" >
                </param>
            </params>
        </url-rule>
    </url-rule-set>
</config>
```
