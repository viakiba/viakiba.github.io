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

#### 源代码
```
代码地址：https://github.com/viakiba/springboot/tree/master/springbootliquibase
```

#### 依赖项
```
<dependency>
			<groupId>org.liquibase</groupId>
			<artifactId>liquibase-core</artifactId>
			<!-- https://github.com/viakiba/springboot/tree/master/springbootliquibase/pom.xml -->
</dependency>
```

#### 配置项
> application.yaml

```
liquibase:
  change-log: classpath:liquibase/master.xml
  enabled: true
  drop-first: false
```

#### change-log

>根据上面的配置项可以确定，此时的change-log也就是master.xml </br>
此时，master.xml 作为管理文件，使用 <include> / <includeAll> 标签对来进行变更文件引入。

如图：
![证书概览](/images/post/201809/changelog1.png)

##### init-scheml
liquibase 是直接支持sql的执行的，所以初始化可以是如下的形式
```
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-2.0.xsd">

	<changeSet author="initialDB" id="init-schema" dbms="mysql">
        <comment>DB initialization.</comment>
				<!--
					author： 创建人 会记录到 liquibase 的默认数据表 databasechangelog 中
					dbms ： 针对的数据库，如果数据源连接的不是mysql，则此脚本不会被执行。如果不指定 则所有数据库类型都会执行。
				-->

	    <sql>
			<!-- 也就是标准 sql 语句 -->
			create table if not exists NUSER (
				id bigint auto_increment unique,
				created_date timestamp,
				last_modified_date timestamp,
				last_modified_user bigint,
				primary key (id)
			);
			create index last_modified_user_index on NUSER (last_modified_user);
	    </sql>
    </changeSet>
</databaseChangeLog>
```

也可以是标签的形式
```
<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
        http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.4.xsd">
    <changeSet id="init-schema" author="initialDB"  dbms="mysql">
        <comment>DB initialization.</comment>
        <createTable tableName="NUSER">
            <column name="id" type="bigint" autoIncrement="${autoIncrement}">
                <constraints primaryKey="true" nullable="false"/>
            </column>
            <column name="created_date" type="timestamp">
                <constraints  nullable="false"/>
            </column>
            <column name="last_modified_date" type="timestamp">
                <constraints  nullable="false"/>
            </column>
            <column name="last_modified_user" type="bigint">
                <constraints  nullable="false"/>
            </column>

        </createTable>

        <modifySql dbms="mysql">
            <append value="ENGINE=INNODB DEFAULT CHARSET utf8mb4 COLLATE utf8mb4_general_ci"/>
        </modifySql>
    </changeSet>
</databaseChangeLog>
```


##### changelog

###### 20181112

###### 20181113



## 总结
