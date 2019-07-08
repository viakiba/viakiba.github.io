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
liquibase ：https://github.com/liquibase/liquibase
参考： http://www.liquibase.org/documentation/index.html
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

					sql 标签： 里面
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
				<!-- 注释 创建表的说明 -->
        <comment>DB initialization.</comment>
				<!-- 创建表的标签 -->
        <createTable tableName="NUSER">
            <column name="id" type="bigint" autoIncrement="true">
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
        <!--
				<!-- 在 createTable 标签生成的SQL后面 追加 SQL 语句 -->
				<modifySql dbms="mysql">
            <append value="ENGINE=INNODB DEFAULT CHARSET utf8mb4 COLLATE utf8mb4_general_ci"/>
        </modifySql>
				 -->
    </changeSet>
</databaseChangeLog>
```

##### master 文件使用 include 标签引入

```
<?xml version="1.0" encoding="utf-8"?>
<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
        http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.4.xsd">

    <include file="classpath:liquibase/init-schema.xml" relativeToChangelogFile="false"/>
</databaseChangeLog>
```

liquibase 会产生两个 数据表用于记录当前的数据表 结构情况：
* databasechangelog
* databasechangeloglock

##### 启动
此时服务器启动 后，数据库会出现 databasechangelog / databasechangeloglock / NUSER 三个文件。

#### 常用操作标签
##### 20181112
操作字段
```
<!-- 新增 -->
<changeSet author="viakiba" id="1" dbms="mysql">
		<comment>Add region column in perf_test table.</comment>
		<addColumn tableName="NUSER">
			<column name="region" type="varchar(255)" />
		</addColumn>
</changeSet>
<!-- 更改字段名称 -->
<changeSet author="viakiba" id="12" dbms="h2">
	    <renameColumn tableName="NUSER"
                 oldColumnName="region" newColumnName="region_new"/>
</changeSet>
<!-- 更改字段类型 -->
<changeSet author="viakiba" id="14" dbms="h2">
	<modifyDataType tableName="NUSER" columnName="region" newDataType="varchar(100)"/>
</changeSet>
```

操作条件标签
```
<?xml version="1.0" encoding="UTF-8"?>

<databaseChangeLog
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog/1.8"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog/1.8
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-1.8.xsd">
    <preConditions>
        <dbms type="oracle" />
        <runningAs username="SYSTEM" />
    </preConditions>
    <changeSet id="1" author="bob">
        <preConditions onFail="WARN">
            <sqlCheck expectedResult="0">select count(*) from oldtable</sqlCheck>
        </preConditions>
        <comment>前置条件通过会执行删表操作 expectedResult 是 SQL 执行的预期值</comment>
        <dropTable tableName="oldtable"/>
    </changeSet>
</databaseChangeLog>
```
这些标签还有很多没有列出来，可以查看 [文档](http://www.liquibase.org/documentation/index.html)进行业务需求的适应，比如区分 生产与测试等环境 使用 [context](http://www.liquibase.org/documentation/contexts.html) 标签。比如
[配置参数](http://www.liquibase.org/documentation/changelog_parameters.html),这个看例子很好理解使用，而且使用很频繁，就不在代码举例了。

##### 基于 Spring 的传统项目

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xmlns:mvc="http://www.springframework.org/schema/mvc"
	   xmlns:context="http://www.springframework.org/schema/context"
	   xmlns:cache="http://www.springframework.org/schema/cache"
	   xmlns:aop="http://www.springframework.org/schema/aop"
	   xmlns:tx="http://www.springframework.org/schema/tx" xmlns:util="http://www.springframework.org/schema/util"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
		http://www.springframework.org/schema/mvc
		http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context-4.3.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.3.xsd
		http://www.springframework.org/schema/cache http://www.springframework.org/schema/cache/spring-cache-4.3.xsd
		http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.3.xsd">
	<!-- 开启注解配置 -->
	<context:annotation-config />

	<!-- Use spring servlet for all requests, including static resources -->
	<mvc:default-servlet-handler/>

	<!-- 开启 @MVC annotations（注解配置） -->
	<mvc:annotation-driven />

	<!-- 开启 @Controller, @Service... annotations（注解配置） com.yuncanting下的所有层级的包都会被扫描 -->
	<context:component-scan base-package="com.yuncanting"/>

	<!-- 设置加载properties 配置文件的方式-->
	<util:properties id="settings" location="classpath*:param.properties"></util:properties>

	<!-- properties配置文件的位置 -->
	<bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
	    <property name="locations">    
	        <list>   <!-- 可以多个 -->
	            <value>classpath*:param.properties</value>
	        </list>    
	    </property>    
	</bean>

	<!-- 加载properties key之对应的参数值 -->
    <bean id="pooledDataSource" class="com.alibaba.druid.pool.DruidDataSource">
    	<property name="url" value="${db.jdbcurl}" />
     	<property name="username" value="${db.username}" />
     	<property name="password" value="${db.password}" />
		<property name="driverClassName" value="${db.driverClassName}"/>
    </bean>

	<!-- 引入 liquibase ****  -->
	<bean id="liquibase" class="liquibase.integration.spring.SpringLiquibase">
      <property name="dataSource" ref="myDataSource" />
      <property name="changeLog" value="classpath:db-changelog.xml" />
      <!--
      contexts specifies the runtime contexts to use.
      -->
      <property name="contexts" value="test, production" />
 </bean>

	<!-- session 工厂 指定mybatis的配置文件的位置-->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="configLocation" value="classpath:mybatis.xml"></property>
		<property name="dataSource" ref="pooledDataSource" />
	</bean>
	<!-- 指定 dao 接口的位置 避免实体类的编写-->
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="basePackage" value="com.yuncanting.dao" />
		<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
	</bean>
	<!-- 事务管理器 -->
    <bean id="dataSourceTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    	<property name="dataSource" ref="pooledDataSource"></property>
    </bean>

	<!-- 指定切面方法的事务方式 -->
    <tx:advice id="tx" transaction-manager="dataSourceTransactionManager">
    	<tx:attributes>
    		<tx:method name="query*" read-only="true"/>
			<tx:method name="*" rollback-for="Throwable"/>
    	</tx:attributes>
    </tx:advice>

	<!--事务切面 指定那些方法进行事务管理-->
    <aop:config>
    	<aop:pointcut expression="execution(* com.yuncanting.service.*.*(..))" id="serviceOperation"/>
    	<aop:advisor advice-ref="tx" pointcut-ref="serviceOperation"/>
    </aop:config>
</beans>
```




## 总结
liquibase 总之特别适合数据库结构变化迭代较快的开发场景，而且数据库操作方式支持很晚上，目前上面只是举了几个简单的例子，其他项目中需要的操作基本都可以找到，使用的IDE是IDEA时是可以提示的，而且即使没有相关的标签操作，也可以直接写SQL语句操作。
