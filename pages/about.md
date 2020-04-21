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

<center>
    <h1>黄鹏</h1>
    <div>
        <span>
            <a href="tel:18037650338">18037650338</a>
        </span>
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
        <span>
            <a href="mailto:viakiba@gmail.com">viakiba@gmail.com</a>
        </span>
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
        <span>
            <a href="https://github.com/viakiba">viakiba</a>
        </span>
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
        <span>
            <a href="https://blog.viakiba.cn">Blog</a>
        </span>
    </div>
</center>

## 教育经历

- 本科 河南科技大学 物联网工程专业(计算机系) 2013.9~2017.6

## 技能清单

- 主要使用 Java 进行开发工作，偶尔也会学习使用 Python 写工具解决一些重复性的问题。
- 常用 Guava 工具库来提高开发效率，也会使用 Lamada表达式使代码更简洁。
- 在游戏开发时期网络IO使用 Mina框架，负责新需求的开发，比如 开挂消息限频，活动，礼物开发。
- 常使用数据库 MySQL, Redis。
- 使用 Spring, Eureka, Feign, Ribbon, Zuul, Hystrix, MyBatis 等框架，完成企业级的系统实现并商用。
- 了解 Html, CSS, JavaScript, Jquery, EasyUI 等前端技术。
- 熟悉 GIT, SVN, IDEA, Eclipse, Nginx, Jenkins, Bash 等工具。
- 了解 Docker 容器技术。

## 工作经历

- 开心网，开瑞工作室，开发工程师，2019.11~至今
- 雷森科技发展有限公司，研发部，Java工程师，2018.4~2019.11
- 北京龙马游戏，研发部，Java开发，2017.7~2018.10

## 项目经历

- TSM平台
  - 背景：TSM平台包含两部分，一部分是可信服务下的环境初始化实现 简称 SEI ，一部分是个人化操作的业务实现 简称 SP 。例如在手机中安装一个地铁卡实现刷卡功能，安装门禁卡的过程就包含上述两个过程。目前接入该系统的终端厂商有 华为，联想，出门问问等。
  - 描述：
      - 接入VISA的VTS系统时，按照JWS 与 JWE 规范实现数据的完整性校验已经加密保护。
      - 根据VISA的VTS系统定义的秘钥体系，使用很多加密技术完成接口调用，比如 AES / 3DES / RSA 等，并完成PCI DSS 系统安全认证。
      - 该部分职责：GPAC / 安全域/ 包 下载/删除 与 密钥更新 （洗卡操作），负责KMS密钥系统对接 等业务，并且负责该服务对应管理系统的开发。
      - 负责服务化改造，改用注册中心进行服务发现，使用feign进行服务间调用。
  - 补充
    - 华为交通卡接入之后进行过压力测试，测试工具是我负责选型与实现的脚本。测试目标是每秒事务数要达到400TPS。
    - 因为最终工具提供给华为需要他们操作进行观察，结合事务数目标，选用 Ngrinder 开源项目进行构建，在这个项目基础上 接入mysql数据源能力（那个时候这个项目不支持mysql只支持H2)，并且新增了秘钥管理页面，这个项目使用 SpringData / FreeMarker 等技术实现的web系统。
    - 脚本实现使用Groovy引擎，编写脚本调用jar，jar包完成压测逻辑的业务实现。
    - 了解学习Jcommand库实现命令能力，了解学习liquibase变更追踪库。
    - 遇到过一个性能问题，就是脚本执行时，压测服务器的并发很低，排查发现是logback日志打印消耗了很多时间。解决方案：删除不必要的日志 日志打印采用异步的方式，结果性能提高一倍以上，使得有限的服务器资源完成压测目标。

- 东升智慧园区系统
  - 背景: 东升科技园园区一卡通升级改造，联合阿里重新帮他们构建一卡通交易系统（采用阿里IOT的账户体系）。目前进入测试阶段。主要负责 POSApp的接口和业务对接以及ZUUL网关路由实现。项目采用微服务体系。
  - 描述：
    - 使用 SpringBoot (Eureka，Feign, Ribbon, Redis, MyBatis) 进行业务实现。
    - ZUUL：使用 ZuulFilter 与 RequestContext 完成签名验证与必要参数的校验。
    - ZUUL：路由采用 配置文件方式 zuul.routes 实现。
    - 服务发现使用 Eureka 注册中心保证服务的高可用。
    - POSApp的接口业务实现主要负责了核销券码，订单验证上传撤销等逻辑。
    - POSApp与交易核心模块的对接使用feign进行的外部调用
    - 使用 Hystrix 实现服务不可用时快速失败，避免服务雪崩。
  - 补充
    - 初始化缓存模块的封装实现：缓存都是放在了Redis中，项目中使用的在公共模块负责实现了缓存实现的定义封装，只需实现定义的方法即可。

- 猫游记
  - 背景：养成类即时战斗游戏，主要使用的技术 Mina2 + Hibernate (Lombok+POI+Groovy+BSF)   (STS+JDK1.7+ANT)。
  - 描述：
    - 参与游戏的任务模块与活动模块（答题活动）的研发工作。 
    - 移动端手游，采用长连接，数据格式采用字节读取约定，协议采用XML进行定义，工具进行类的生成（消息BEAN代码生成使用volocity模板引擎）。
    - 持久层使用Hibernate框架,缓存框架采用公司自研库。
    - 大量使用单例与责任链模式，策划配置采用POI技术与BSF技术。
    - 依赖lombok,精简修改消息类代码生成工具。

- 天书奇谈
  - 背景：天书奇谈是人人游戏平台一个历史较长的游戏，我从入职公司，这个游戏一直由我来整体接手并负责，在此期间，主要为游戏添加了 二级保护密码，外挂级别梯度调控（避免服务器负载过大），以及游戏活动 乾坤画匣，使者来朝，中元节，国庆节等活动。并完成日常bug的修复工作。并负责游戏GM后台的日常维护（servlet/jsp）,例如奖品发放，异常用户维护，服务器停服起服等。主要使用 Mina ， Hibernate 。
  - 描述：
    - 采用Socket（IO框架Mina1）进行数据交互。数据存储使用Hibernate。
    - 大量使用单例与责任链模式。
    - 策划配置采用POI技术与BSF技术。线上数据异常，项目使用Groovy来进行数据维护。    
