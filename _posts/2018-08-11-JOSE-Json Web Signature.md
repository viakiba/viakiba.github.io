---
layout: post
title: JOSE-Json Web Signature
categories: [JWS]
description: Json Web Signature
keywords: JOSE, Json Web Signature, JWS, Java
tags: JWS, JOSE, Json Web Signature, Java
---

JOSE-Json Web Signature (RS256 算法)

## 代码与参考
```
  代码：
    https://github.com/viakiba/viakiba/tree/master/josedemo
  参考：
    https://en.wikipedia.org/wiki/JSON_Web_Signature
    https://bitbucket.org/b_c/jose4j/wiki/Home
    https://tools.ietf.org/html/rfc7515
```

## 背景
>最近项目组在做VISA信用卡支付相关的业务。在对接VISA系统时，卡信息加密数据的格式按照Json Web Encryption 规范 ,完整性控制 按照 JSON Web Signature 规范也就是签名数据格式规范。在实现过程中由于不熟悉这个规范而且国内也很少提及这个规范，在此间遇到很多问题。借此机会，把我知道的这个实现流程做一个文档，以供大家参考。希望对大家有所帮助。

> JWS (Json Web Signature) 是一种以高度可序列化的机器可读格式确保信息完整性的方法。这意味着它是信息，同时证明自签署以来信息没有改变。它可用于从一个网站向另一个网站发送信息，尤其是针对网络上的通信。它甚至包含针对URI查询参数等应用程序优化的紧凑形式。

>JWS (Json Web Signature) 只保证信息的完整性与确认是否被篡改的能力。不具备加密属性，加密实现 比如 Json Web Encryption.

## 数字信息的签名与验签

> 数字签名（又称公钥数字签名、电子签章）是一种类似写在纸上的普通的物理签名，但是使用了公钥加密领域的技术实现，用于鉴别数字信息的方法。一套数字签名通常定义两种互补的运算，一个用于签名，另一个用于验证。签名其实质，是先对待签名的数据取Hash值， 然后使用私钥对hash值进行加密的过程。验签，就是使用公钥ui签名的结果进行解密的过程。

> 数字签名，就是只有信息的发送者才能产生的别人无法伪造的一段数字串，这段数字串同时也是对信息的发送者发送信息真实性的一个有效证明。

>数字签名是非对称密钥加密技术与数字摘要技术的应用。

> 参考此文的 1-10 ：https://blog.csdn.net/xiangwanpeng/article/details/70834060

> 10之后为中间人攻击问题，本文不做考虑，签名和验签无法解决中间人攻击问题。

## JWS 算法 常见的列表

| Digital Signature or MAC Algorithm | JWS "alg" Parameter Value |
| ------ | ------ |
| HMAC using SHA-2 | HS256, HS384 and HS512 |
| RSASSA-PKCS1-V1_5 Digital Signatures with with SHA-2 | 	RS256, RS384 and RS512 |
|Elliptic Curve Digital Signatures (ECDSA) with SHA-2 | ES256, ES384 and ES512 |
|RSASSA-PSS Digital Signatures with SHA-2 | PS256†, PS384† and PS512† |
|Unsigned Plaintext | none |

## RS256 （SHA256withRSA）实现
>在这里借助java里的JOSE库来实现，因为JOSE库的实现符合 RFC7515 规范。并且使用java自带的实现代码进行分布验证。

### 签名流程图如下：
![JWS](/images/post/201808/jwe_RS1_5-JWS.png)

接下来的实现思路是： 使用JOSE库对数据进行签名，然后得到的结果，在不借助JOSE库的情况下对得到的结果进行数据的进行完整性验证。从而确认上述图片中的逻辑。

### 依赖：

```
<dependency>
    <groupId>org.bitbucket.b_c</groupId>
    <artifactId>jose4j</artifactId>
    <version>0.6.4</version>
</dependency>
<dependency>
    <groupId>com.nimbusds</groupId>
    <artifactId>nimbus-jose-jwt</artifactId>
    <version>4.34.2</version>
</dependency>
```

### 公私钥信息
PUBLIC_MODULUS 与 PUBLIC_EXPONENT 需要分享给接收者。分享形式自己定义即可。这两个信息一般是X.509证书里的核心信息。一般会作为证书（HTTPS TLS/SSL 证书）分享给接收者。
```
PUBLIC_MODULUS = "00c6a2d057f4067dd43a14a4db8589039f817e97aaf89191fdb0db5cebb0ea6027224ae1226626a73cf15fc8d4defb264a90568abd24d10182dbe191b8eae49c21900258a35761208f4146df4238f1dd3fb9ae47e50653656919b40a3edfb4d1fae7b111b3cd78c913ee60ce50b3e749d693f9bf7577d1ef0001f8b04a6f5a10b07106efcca1c69f280e152327f6744ba7d0e11dbbb8031f35c8838ea49c6fa9ac2f6992c605f3272b0edf445116b6d27606d471a026c4bf1aa3382c8ab20d2d161f6d188f415006deb8529738aa2c2ce343815e59bffc1ba23293d5aae06a523d30692d96fa669794733a5fb280bc20a0f08aada2e8d5a47e4104ea4db65cd11f";
PUBLIC_EXPONENT = "10001";
PRIVATE_EXPONENT = "00a2eb7d88f63cb0cdf60962a22ee78f522f8b1e68fbd1a1f57b2ea10b2ba340d4383b4466cb741ead4ca8ac7774a077eaa67264fef808797dd44d3211828f9943a9f352b23e840a899517c51c72ca6616d37c0fb9d83364b50c80effa5bcfda7e39b4b0f951a924fbb5042f945fca6f7491104229ddea116667378b98b1b62482afcc3633f50602e4f3d2e8a50e14636518fed11f7bc79a9ad9ffd40c1a69a423de5de3316601a344efb4b4a86944fd13d1d4409bc4a1a57238f8d2bf9c756349c5e8c9cc51832661eca3fc51ecffd7a42273262f5e3c6405284fff565ccc67e348dc94cfd4043dc670497c39e967c4a4d5fc1458ac1965d92068fb951a7729d1";
```

我们使用证书进行分享给VISA的，整个证书的根证书是通过线下的方式分享给VISA。这个证书通过根证书进行签发，所以在一定意义上保证了分享的证书的信任度。只要根证书不泄露，那个分享的证书可以认为是可信的。

### JOSE 的签名实现

```
//使用 JOSE 库对数据进行签名
String message = "这是RSA1_5加密算法的jwe示例。";
JsonWebSignature senderJws = new JsonWebSignature();
senderJws.setAlgorithmHeaderValue(AlgorithmIdentifiers.RSA_USING_SHA256);//设置签名算法
senderJws.setHeader("iat",String.valueOf(System.currentTimeMillis()));
senderJws.setPayload(message);//设置待签名数据

KeyFactory keyFactory = KeyFactory.getInstance("RSA");
RSAPrivateKeySpec rsaPrivateKeySpec = new RSAPrivateKeySpec(new BigInteger(JoseConstants.PUBLIC_MODULUS,JoseConstants.RSA_KEY_RADIX),
       new BigInteger(JoseConstants.PRIVATE_EXPONENT,JoseConstants.RSA_KEY_RADIX));
PrivateKey privateKey = keyFactory.generatePrivate(rsaPrivateKeySpec);//构建签名需要的私钥

senderJws.setKey(privateKey);//把私钥设置进去
String compactSerialization = senderJws.getCompactSerialization();//签名并得到结果
//compactSerialization 就是JWE规范的数据格式下的签名结果，
//包括三部分，使用 . 分割，
//第一个是 header 主要包含签名算法，签名时的时间等 可以自定义 使用 setHeader 设置。
//第二部分是是签名的明文信息。 也就是上面的message
//第三部分就是签名的得到的结果，这三部分就是发送给接收者的数据，接收者会根据这个字符串使用实现得到的接收者的公钥信息进行验证。
System.out.println(compactSerialization);
```
### 分步验证
分步验证作为上述流程的逆向实现与验证。使用的字符串 就是 compactSerialization （上面生成的）。

```
//代码验证签名
String[] split = compactSerialization.split("\\.");
String encodeHeader = split[0];
String encodeData = split[1];
String encodeSign = split[2];

String finalData = split[0] +"."+ split[1];//拼接实际的签名数据 包括header部分，heder一般会包括时间戳信息
boolean verify = verify(finalData, encodeSign, JoseConstants.RSA_KEY_RADIX, JoseConstants.PUBLIC_MODULUS, JoseConstants.PUBLIC_EXPONENT,"SHA256withRSA");
```
对于 verify 方法的实现如下：
```
/**
 * @description 验证签名
 * @date: 21:52 2018/7/28
 * @params [plainData 带签名的信息 , sign 签名结果 , radix 证书参数进制, publicModulus 公钥的模, privateExponent 私钥指数, alg 算法名称]
 * @return java.lang.String
 * @author viakiba
 */
public static boolean verify(String plainData, String sign, int radix, String publicModulus, String publicExponent, String alg) throws Exception {
    KeyFactory keyFactory = KeyFactory.getInstance("RSA");
    RSAPublicKeySpec keySpec = new RSAPublicKeySpec(new BigInteger(publicModulus, radix), new BigInteger(publicExponent, radix));
    //构建公钥信息
    PublicKey publicKey = keyFactory.generatePublic(keySpec);
    final Signature signature = Signature.getInstance(alg);
    signature.initVerify(publicKey);
    //getBytesAscii 其实就是获取字符串的 ascII 字节。
    signature.update(StringUtil.getBytesAscii(plainData));
    // 对签名结果先做 Base64Url 解码
    byte[] decode = Base64Url.decode(sign);
    return signature.verify(decode);
}
```

### 使用JOSE库进行验证
compactSerialization 已经包含了使用的算法信息以及签名的原文数据，产生的header 以及签名结果，所以这里只要设置进去公钥信息就可以验证信息的完整性了。
```
JsonWebSignature senderVerifyJws = new JsonWebSignature();
senderVerifyJws.setCompactSerialization(compactSerialization);
RSAPublicKeySpec keySpec = new RSAPublicKeySpec(new BigInteger(JoseConstants.PUBLIC_MODULUS, JoseConstants.RSA_KEY_RADIX),
        new BigInteger(JoseConstants.PUBLIC_EXPONENT, JoseConstants.RSA_KEY_RADIX));
PublicKey publicKey = keyFactory.generatePublic(keySpec);
senderVerifyJws.setKey(publicKey);
boolean b = senderVerifyJws.verifySignature();
System.out.println(b ? "签名被认可":"签名不被认可");
```

## 结束语

> 正如背景里所属，这些常用于金融等领域的消息传递。其也是JWT（Json Web Token 的基础）。
