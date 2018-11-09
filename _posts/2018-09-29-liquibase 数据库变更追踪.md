---
layout: post
title: liquibase 数据库变更追踪
categories: [liquibase, 数据库]
description: Liquibase是一个用于跟踪、管理和应用数据库变化的开源的数据库重构工具。
keywords: liquibase, 数据库
---

## 概述

liquibase是一个与具体数据库独立的追踪、管理和应用数据库Scheme变化的工具。

## 特性

* 支持多种数据库类型 如MySQL, PostgreSQL, Oracle, Sql Server, DB2, H2等
* 支持XML、YAML、JSON 与 SQL格式
* 支持上下文相关逻辑
* 支持集群安全的数据库升级
* 生成数据库变更文档
* 生成数据库“diff”
* 穿透构建流程，可根据应用需要嵌入到应用中
* 自动生成SQL脚本，供DBA进行代码审查


* 支持像 Create Table 和 Drop Column 这样的简单命令
* 支持像 Add Lookup Table 和 Merge Columns 这样的复杂命令
* 执行 SQL
* 支持生成与管理回滚逻辑


* 支持多种运行方式，如命令行、Spring集成、Maven插件、Gradle插件等
* 并不能适用于带数据转移，不支持存储过程。

## 实践

### SpringBoot 集成

```
代码地址：
```

pom文件引入liquibase库
```
<dependency>
			<groupId>org.liquibase</groupId>
			<artifactId>liquibase-core</artifactId>
		</dependency>
```



## 总结
