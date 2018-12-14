---
layout: post
title: ReloadableResourceBundleMessageSource：实现WEB端国际化方案
categories: [Spring, I18N]
description: ReloadableResourceBundleMessageSource：实现WEB端国际化方案
keywords: ReloadableResourceBundleMessageSource, I18N
---

## 概述
>前一段着手修改一个开源的压测平台供内部使用，在修改的过程中遇到一个网页国际化的实现方式，感觉很有用，于是就着手写了一个demo实现了一个简易版。

>>代码见 https://github.com/viakiba/springboot/tree/master/springbooti18n

## 环境

```
SpringBoot 2.1.1.RELEASE
spring-boot-starter-thymeleaf 2.1.1.RELEASE

详情见 pom 文件：
  https://github.com/viakiba/springboot/blob/master/springbooti18n/pom.xml

基于maven构建

国际化 基于 ReloadableResourceBundleMessageSource 实现， 使用 thymeleaf 3.0.11 进行页面渲染验证 （也可以是 jsp / freemarker 等）

```

## 配置

```
application.properties 不做配置。基于 javaConfig 进行配置
配置文件见 ： https://github.com/viakiba/springboot/blob/master/springbooti18n/src/main/java/org/viakiba/i18n/I18nConfig.java

I18nConfig 需要 继承 WebMvcConfigurerAdapter抽象类 与 ApplicationContextAware 接口。
```

### messageSource
```
//注入 messageSource，name 需要指定名字且为 messageSource ：
@Bean(name = "messageSource")
public MessageSource setReloadableResourceBundleMessageSource(){
   ReloadableResourceBundleMessageSource reloadableResourceBundleMessageSource = new ReloadableResourceBundleMessageSource();
   reloadableResourceBundleMessageSource.setDefaultEncoding("UTF-8");
   reloadableResourceBundleMessageSource.setCacheSeconds(0);
   reloadableResourceBundleMessageSource.setUseCodeAsDefaultMessage(true);
   // 放在resouce 下的 i18n 文件夹 并以 message开头 下划线分割 以 国家/语言代码 结尾
   reloadableResourceBundleMessageSource.setBasename("classpath:/i18n/messages");
   return reloadableResourceBundleMessageSource;
}

//设置默认的 locale 标识

//其实这一步会写进 cookie 
@Bean
public LocaleResolver localeResolver() {
    CookieLocaleResolver cookieLocaleResolver = new CookieLocaleResolver();
    cookieLocaleResolver.setDefaultLocale(Locale.ENGLISH);
    return cookieLocaleResolver;
}


//设置一个 locale 拦截器
@Bean
public LocaleChangeInterceptor localeChangeInterceptor() {
    LocaleChangeInterceptor lci = new LocaleChangeInterceptor();
    lci.setParamName("lang");
    return lci;
}
@Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(localeChangeInterceptor());
}
//这样就可以在 url 中 根据 lang 进行修改 locale 标识 例如：
//http://127.0.0.1:8080/?lang=en_US

```

### thymeleaf 配置

```
//注入 applicationContext：

private ApplicationContext applicationContext;

@Override
public void setApplicationContext(ApplicationContext applicationContext) {
    this.applicationContext = applicationContext;
}

//注入 ViewResolver SpringTemplateEngine ITemplateResolver
@Bean
public ViewResolver viewResolver() {
   ThymeleafViewResolver resolver = new ThymeleafViewResolver();
   resolver.setTemplateEngine(templateEngine());
   resolver.setCharacterEncoding(UTF8);
   return resolver;
}

public SpringTemplateEngine templateEngine() {
   SpringTemplateEngine engine = new SpringTemplateEngine();
   engine.setTemplateResolver(templateResolver());
   return engine;
}

public ITemplateResolver templateResolver() {
   SpringResourceTemplateResolver resolver = new SpringResourceTemplateResolver();
   resolver.setApplicationContext(applicationContext);
   resolver.setPrefix("/templates/");
   resolver.setSuffix(".html");
   resolver.setTemplateMode(TemplateMode.HTML);
   return resolver;
}

// thymleaf 配置在 SpringBoot2.1.1 中与 1.5.x配置 有不同 1.5.x下见
https://github.com/viakiba/viakiba/tree/master/thymeleaf3-spring-helloworld
```
