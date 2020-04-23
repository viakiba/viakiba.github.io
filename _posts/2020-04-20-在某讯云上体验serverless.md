---
layout: post
title: 在某讯云上体验serverless
categories: [serverless,serverlessDB]
description: 在某讯云上体验serverless
keywords: serverless,serverlessDB
---

### 介绍

```txt
    serverless 都说好几年了，最近试了一下腾讯云上的 serverless。
    包含 函数书写 和 DB 操作 
    serverless 概念：
        https://cloud.tencent.com/document/product/583/30558
    serverless Python：
        https://cloud.tencent.com/document/product/583/11061
```

### 实践

- 创建 Api 密钥
```
https://console.cloud.tencent.com/cam/capi
```

- 创建 serverless DB 

```
在线工具
    https://console.cloud.tencent.com/api/explorer?Product=postgres&Version=2017-03-12&Action=DescribeServerlessDBInstances&SignVersion=
创建指引
    https://cloud.tencent.com/document/product/409/42847

注意 serverlessDB请按照指引开启外网访问 postgreSQL客户端就可以直接连接管理 比如 navicat

创建表
    CREATE TABLE "public"."paste" (
        "uuid" varchar(100) COLLATE "pg_catalog"."default" NOT NULL,
        "appid" varchar(255) COLLATE "pg_catalog"."default",
        "content" bytea,
        "type" int4,
        "createTime" timestamp(6),
        CONSTRAINT "paste_pkey" PRIMARY KEY ("uuid")
        )
        ;
```

![创建在线工具的界面](/images/post/202004/1.png)

- 创建函数

```
创建的页面如下
    https://console.cloud.tencent.com/scf/list?rid=8&ns=default

如下图所示 进行选择
```
![创建在线工具的界面](/images/post/202004/2.png)


添加环境变量
![创建在线工具的界面](/images/post/202004/3.png)

```
DB_DEFAULT DB1
DB_DB1_DATABASE 数据库名称
DB_DB1_PASSWORD 密码
DB_DB1_USER 用户名
DB_DB1_PORT 端口
DB_DB1_HOST 地址
```
![创建在线工具的界面](/images/post/202004/4.png)

- 代码

```python
# -*- coding: utf8 -*-
import json
from os import getenv
import psycopg2
from serverless_db_sdk import database
# psycopg2 连接  postgreSQL 的库
def main_handler(event, context):
    print('Start Serverlsess DB function')
    print ('------------------------------------------------')
    print (json.dumps(event) +'  '+ json.dumps(context))
    print ('--------------------------------------------')
    
    if("POST" == event['httpMethod']):
        conn=psycopg2.connect(database=getenv("DB_DB1_DATABASE"),user=getenv("DB_DB1_USER"),password=getenv("DB_DB1_PASSWORD"),host=getenv("DB_DB1_HOST"),port=getenv("DB_DB1_PORT"))
        cursor=conn.cursor() #创建指针对象
        sql = 'SELECT * FROM paste '
        cursor.execute( sql )
        myresult = cursor.fetchall()
        result = ''
        for x in myresult:
            result = result + str(x[1]) + '$$$'
            return result
    return "hello world"
```

### 开启触发

![创建在线工具的界面](/images/post/202004/5.png)