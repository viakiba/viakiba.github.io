---
layout: post
title: docker 安装 WordPress
categories: [docker]
description: docker 安装 WordPress
keywords: docker
---

### 安装

#### 数据库配置
```sql
-- wordpress 链接的数据库是宿主机本地安装的（非docker容器）
-- 先链接mysql创建数据库 wordpressdb
-- 修改数据库访问网段  
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password*' WITH GRANT OPTION;
flush PRIVILEGES;
```

#### 安装

```shell
# 本地86端口映射到容器的80端口
#参考 https://hub.docker.com/_/wordpress/ 
#https://www.jianshu.com/p/3e1fd311ba87
#http://www.voidcn.com/article/p-nyrhtuzl-d.html
# WORDPRESS_DB_HOST 地址 需要携程宿主机的局域网地址才可访问 如果写127.0.0.1 则容器内找的是容器自己的127.0.0.1
docker run -d --name wordpress -e WORDPRESS_DB_NAME=wordpressdb -e WORDPRESS_DB_HOST=192.168.1.121:3306 -e WORDPRESS_DB_USER=root -e WORDPRESS_DB_PASSWORD=password123456 -p 86:80 wordpress:php7.2
```

#### nginx 配置

```shell
#参考 
server {
        server_name wordpress.demo.cn;
	listen 443;
        ssl_certificate  cert/wordpress.demo.cn/wordpress.demo.cn.pem;
        ssl_certificate_key  cert/wordpress.demo.cn/wordpress.demo.cn.key;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1.2;
        ssl_prefer_server_ciphers on;

# https://legolasng.github.io/2018/06/26/nginx-reverse-proxy-wordpress/

	location / {
            tcp_nodelay     on;
            proxy_set_header Host            $host;
            proxy_set_header X-Real-IP       $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            # 向后端服务传递请求协议类型
            proxy_set_header X-Forwarded-Proto $scheme;
	        proxy_pass http://127.0.0.1:83;
        }
    }

```

### 部署一个第三方主题

下载：
```
https://www.themepark.com.cn/msjymfwordpresszttyb.html#cschak
```
参考：
        外观 -> 主题 -> 添加 -> 上传主题 (按钮)
![主题压缩包](/images/post/201910/homemagic_free.zip)

参考：
        插件 -> 安装插件 -> 上传插件（按钮）
        https://www.themepark.com.cn/wordpressyjdrsjcjxz.html

![插件导入](/images/post/201910/one-click-demo-import.zip)

插件导入内容：
        参考 ： https://www.themepark.com.cn/xbbwordpressztdryssjjc.html
        外观 -> 导入演示数据 -> 导入演示数据