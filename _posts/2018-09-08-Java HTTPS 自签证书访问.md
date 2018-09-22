---
layout: post
title: Java Https自签证书访问
categories: [Https,Java]
description: Java Https自签证书访问
keywords: Https, Java, 自签证书
---

## 缘起
>由于所在项目中根据安全的考虑，各服务之间的调用都必须启用 Https 。由于是内部间的调用，所以可以使用自签的证书来完成此要求。此时牵涉到一个问题就是自签证书的并不在Java的CA证书库下。所以在访问此链接是会出现类似的如下错误：

```
javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path validation failed: java.security.cert.CertPathValidatorException:
```

## 环境准备

自签发证书可以借助 [OpenSSL](https://www.openssl.org/) 来完成：

```
根证书生成:
  1.1 生成CA的key
  openssl genrsa -des3 -out ca.key 4096
  1.2 生成CA的证书
  openssl req -new -x509 -days 365 -key ca.key -out ca.crt
    会提示输入server.key的密码
      开始输入Country Name：CN
      State or Province Name：BeiJing
      Locality Name：BeiJing
      Organization Name：Laser (公司英文名)
      Organizational Unit Name：这个可以忽略 （组织单位名称）
      Common Name：域名 （关键参数）
      Email Address：可忽略
      A challenge password：可忽略
      An optional company name ： 可忽略

Https证书生成（由根证书签发）
  2.1 生成证书的key
  openssl genrsa -des3 -out myserver.key 4096
  2.2 生成证书请求文件 CSR
  openssl req -new -key myserver.key -out myserver.csr
     这一步的信息输入参考 1.2
  2.3 使用第一步的根证书进行签发 得到 crt
  openssl x509 -req -days 365 -in myserver.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out myserver.crt
  2.4 生成 P12 文件 （windows可以直接打开，可以配置进 Spring Boot 直接使用 实现 https）
  openssl pkcs12 -export -clcerts -in .\myserver.cer -inkey .\myserver.key -out myserver.p12
    SpringBoot 配置：
      #server.ssl.key-store=classpath:myserver.p12
      #server.ssl.key-store-password= 生成P12输入的密码
      #server.ssl.keyStoreType=PKCS12
      #server.ssl.enabled-protocols=TLSv1.2
证书链验证
  openssl verify -CAfile .\ca.crt .\myserver.crt

解析 crt
  openssl x509 -noout -text -in .\myserver.crt

解析 csr
  openssl req -noout -text -in myserver.csr

解析 key
  openssl rsa -noout -text -in myserver.key
```

## 解决 Https 访问证书验证问题

### 方案一
把证书加载进java默认证书库
```
  keytool -import -alias ${alias} -keystore ${JAVA_HOME}/jre/lib/security/cacerts -file ${path-to-certificate-file}
    ${alias} 请替换为你想使用的名称，比如我这里是 viakiba
    ${JAVA_HOME} 请替换为你自己的 JAVA_HOME 目录，比如我这里是 D:\software\java\java8\jdk1.8.0_101
    ${path-to-certificate-file} 请替换为你的ca证书路径，比如我这里就是当前目录的 myserver.crt
    此文件的默认密码 changeit
  例如：
  导入
    keytool -import -alias ${viakiba} -keystore D:\software\java\java8\jdk1.8.0_101\jre\lib\security\cacerts -file myserver.crt
  列表
    keytool -list -keystore D:\software\java\java8\jdk1.8.0_101\jre\lib\security\cacerts
  删除
    keytool -delete -keystore D:\software\java\java8\jdk1.8.0_101\jre\lib\security\cacerts
     别名 viakiba
     口令 changeit
```
  如果有多个证书需要加载进证书库比较繁琐，可以只加载根证书就可以完成所有此根证书签发的 Https 证书的信任。即 把 myserver.crt 改问 CA.crt 。某些情况下，这种做法存在很大的风险，如果根密钥泄露，那么对方使用此密钥签发的证书就是不可信的了。所以我们没有使用此方案，但是此方案可以解决问题

### 方案二
把Https证书加入到环境变量
```
  须知：
    jks文件是可以存入多个证书的这个大前提要知道
  导入：
    keytool -keystore http.jks -keypass changeit -storepass changeit -alias leisenserver -import -trustcacerts -file myservcer.crt

生效方式：
    1.在 myservcer.crt 同级目录执行如上命令：
      System.setProperty("javax.net.ssl.trustStore", "C:\\Users\\Administrator\\Desktop\\kms\\http.jks");
      System.setProperty("javax.net.ssl.trustStorePasePassword", "changeit");
      这种侵入性太强，其他jvm的程序也会受到影响。
    2.也可以加入到启动参数中：
        -Djavax.net.ssl.trustStore=C:\\Users\\Administrator\\Desktop\\kms\\http.jks -Djavax.net.ssl.trustStorePassword=changeit
        java -Djavax.net.ssl.trustStore=C:\\Users\\Administrator\\Desktop\\kms\\http.jks -Djavax.net.ssl.trustStorePassword=changeit -jar APP.jar
```
>这种做法可以让单一站点实现 访问不出现 PKIX path validation failed 问题，但是其他正常的域名并不在 http.jks 信任库里，所以此种情况下访问其他域名爱是会出现此问题。丹斯访问其他域名的时候会出现此问题，可以执行的做法,把其他访问的域名证书也在到 http.jks 中。

### 方案三
>放弃证书校验
  https://www.programcreek.com/java-api-examples/javax.net.ssl.TrustManager

### 方案四
  构建 HttpClient 时，加载信任证书，特别是多个不同网址访问时，代码量较大。
>https://www.kancloud.cn/longxuan/httpclient-arron/117507


方案4虽然复杂但是不依赖环境，方案1，2，3比较简单粗暴，但是省事。
