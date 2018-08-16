---
layout: post
title: JOSE-Json Web Encryption
categories: [JWE]
description: JOSE
keywords: JOSE, JWE, Json Web Encryption
---

JOSE-Json Web Encryption (RSA1_5/A128GCM 算法)

## 代码与参考/
```
  代码：
    https://github.com/viakiba/viakiba/tree/master/josedemo
  参考：
    https://tools.ietf.org/html/rfc7516
    https://bitbucket.org/b_c/jose4j/wiki/JWE%20Examples
    https://bitbucket.org/b_c/jose4j/wiki/JWE%20Examples
```

## 背景
> 最近项目组在做VISA信用卡支付相关的业务。在对接VISA系统时，卡信息加密数据的格式按照Json Web Encryption 规范 ,完整性控制 按照 JSON Web Signature 规范也就是签名数据格式规范。在实现过程中由于不熟悉这个规范而且国内也很少提及这个规范，在此间遇到很多问题。借此机会，把我知道的这个实现流程做一个文档，以供大家参考。希望对大家有所帮助。

> JWE (Json Web Encryption) 是一种以高度可序列化的机器可读格式确保信息加密的方法。

> JWE (Json Web Encryption) 保证信息的安全不被截取读取。不具备完整性校验的属性，完整性校验实现 比如 Json Web Signature.

## JWE 算法列表

JWE Key Encryption and Key Agreement （如何对key进行加密 （key是用来加密明文信息的））

| Key Management Algorithm | JWE "alg" Parameter Value |
| ------ | ------ |
| Direct encryption with a shared symmetric key	| dir |
| RSAES-PKCS1-V1_5 key encryption	| RSA1_5 |
| RSAES using OAEP key encryption	| RSA-OAEP and RSA-OAEP-256¶ |
| AES key wrap |	A128KW, A192KW* and A256KW* |
| AES GCM key encryption |	A128GCMKW‡, A192GCMKW*‡ and A256GCMKW*‡ |
| Elliptic Curve Diffie-Hellman Ephemeral Static key agreement using Concat KDF |	ECDH-ES |
| Elliptic Curve Diffie-Hellman Ephemeral Static key agreement using Concat KDF with AES key wrap	| ECDH-ES+A128KW, ECDH-ES+A192KW*,  ECDH-ES+A256KW* |
| PBES2 with HMAC SHA-2 and AES key wrapping |	PBES2-HS256+A128KW, PBES2-HS384+A192KW* and PBES2-HS512+A256KW* |

JWE Content Encryption （加密明文信息的方案）

| Content Encryption Algorithm |	JWE "enc" Parameter Value |
| ------ | ----- |
| Authenticated encryption with AES-CBC and HMAC-SHA2	| A128CBC-HS256, A192CBC-HS384* and A256CBC-HS512* |
| Authenticated encryption with Advanced Encryption Standard (AES) in Galois/Counter Mode (GCM)	| A128GCM‡, A192GCM*‡ and A256GCM*‡ |

## RSA1_5/A128GCM 算法
>接下来的JWE示例，Key加密管理算法采用 RSA1_5 ,明文的加密算法采用 A128GCM 算法。 （A128GCM 是对称加密，使用随机处理的 key对 明文加密，然后使用RSA公钥对 key进行加密。这样做的原因是，非对称加密，消耗的计算资源比较大，所以使用它对key进行加密会节约资源，使用key进行对称加密比较节省计算资源。这样一来，即保证了明文信息的安全，有节约了计算资源。

## 用JOSE库实现

公钥信息是内置到客户端的，所以不会牵涉网络中公钥的传递。

```
PUBLIC_MODULUS = "00c6a2d057f4067dd43a14a4db8589039f817e97aaf89191fdb0db5cebb0ea6027224ae1226626a73cf15fc8d4defb264a90568abd24d10182dbe191b8eae49c21900258a35761208f4146df4238f1dd3fb9ae47e50653656919b40a3edfb4d1fae7b111b3cd78c913ee60ce50b3e749d693f9bf7577d1ef0001f8b04a6f5a10b07106efcca1c69f280e152327f6744ba7d0e11dbbb8031f35c8838ea49c6fa9ac2f6992c605f3272b0edf445116b6d27606d471a026c4bf1aa3382c8ab20d2d161f6d188f415006deb8529738aa2c2ce343815e59bffc1ba23293d5aae06a523d30692d96fa669794733a5fb280bc20a0f08aada2e8d5a47e4104ea4db65cd11f";
PUBLIC_EXPONENT = "10001";
PRIVATE_EXPONENT = "00a2eb7d88f63cb0cdf60962a22ee78f522f8b1e68fbd1a1f57b2ea10b2ba340d4383b4466cb741ead4ca8ac7774a077eaa67264fef808797dd44d3211828f9943a9f352b23e840a899517c51c72ca6616d37c0fb9d83364b50c80effa5bcfda7e39b4b0f951a924fbb5042f945fca6f7491104229ddea116667378b98b1b62482afcc3633f50602e4f3d2e8a50e14636518fed11f7bc79a9ad9ffd40c1a69a423de5de3316601a344efb4b4a86944fd13d1d4409bc4a1a57238f8d2bf9c756349c5e8c9cc51832661eca3fc51ecffd7a42273262f5e3c6405284fff565ccc67e348dc94cfd4043dc670497c39e967c4a4d5fc1458ac1965d92068fb951a7729d1";
```

### 加密流程

![JWE](/images/post/201808/jwe_RS1_5-JWE.png)

### 依赖

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

### 实现代码
#### JOSE 库加密

RSA1_5 需要的公钥建议 是 2048 位。
A128GCM 需要的 key (CEK conten encryption key 可以是 16 byte, IV 可以是 12 byte 都是随机)

```
String message = "这是RSA1_5加密算法的jwe示例。";
JsonWebEncryption senderJwe = new JsonWebEncryption();
//设置明文
senderJwe.setPlaintext(message);
//指定KEY加密算法
senderJwe.setAlgorithmHeaderValue(KeyManagementAlgorithmIdentifiers.RSA1_5);
//指定 RSA1_5 对应的 publicKey
RSAPublicKeySpec rsaPublicKeySpec = new RSAPublicKeySpec(new BigInteger(JoseConstants.PUBLIC_MODULUS,JoseConstants.RSA_KEY_RADIX),
        new BigInteger(JoseConstants.PUBLIC_EXPONENT,JoseConstants.RSA_KEY_RADIX));
KeyFactory keyFactory = KeyFactory.getInstance("RSA");
PublicKey publicKey = keyFactory.generatePublic(rsaPublicKeySpec);
senderJwe.setKey(publicKey);
//指定对明文的加密算法  AES_128_GCM 决定了参数 会存在 key / iv / aad (key/iv 可以随机生成 我这里直接写死成 0填充 byte[] , aad 是由 header 决定的)
senderJwe.setEncryptionMethodHeaderParameter(ContentEncryptionAlgorithmIdentifiers.AES_128_GCM);
//指定加密 plainText 所需的 cek 为了方便 自定义为32byte的0 原则上随机 可以使用 SecureRandom 的 SHA1PRNG 算法随机
senderJwe.setContentEncryptionKey(new byte[32]);
//指定加密 plainText 所需的 salt 为了方便 自定义为12byte的0 随机建议同上
senderJwe.setIv(new byte[12]);
//设置自定义的 header
senderJwe.setHeader("headerCustom","hhh");
String compactSerialization = senderJwe.getCompactSerialization();
System.out.println("加密后的结果：" + compactSerialization);
```

#### JOSE 库解密
```
// 解密对象
JsonWebEncryption receiverJwe = new JsonWebEncryption();
// 设置算法对应的私钥 key
RSAPrivateKeySpec rsaPrivateKeySpec = new RSAPrivateKeySpec(new BigInteger(JoseConstants.PUBLIC_MODULUS,JoseConstants.RSA_KEY_RADIX),
        new BigInteger(JoseConstants.PRIVATE_EXPONENT,JoseConstants.RSA_KEY_RADIX));
PrivateKey privateKey = keyFactory.generatePrivate(rsaPrivateKeySpec);
receiverJwe.setKey(privateKey);
// 设置密文 密文里免包含了加密的算法以及参数 只需设置私钥即可。
receiverJwe.setCompactSerialization(compactSerialization);
// 得到明文
String plaintext = receiverJwe.getPlaintextString();
System.out.println("解密结果是: " + plaintext);
```

#### 分步验证

```
String[] split = compactSerialization.split("\\.");

String headerSerializa = split[0];
String encCek = split[1];
String encIv = split[2];
String encText = split[3];
String encTag = split[4];

//先解密得到cek
byte[] cek = cekDecryption(Base64.decode(encCek), JoseConstants.RSA_KEY_RADIX, JoseConstants.PUBLIC_MODULUS, JoseConstants.PRIVATE_EXPONENT);
//
System.out.println(new String(Base64.decode(headerSerializa)));
//组装密文
byte[] encTextByte = Base64.decode(encText);
byte[] encTagByte = Base64.decode(encTag);

byte[] encTextAll = new byte[encTextByte.length + encTagByte.length];
System.arraycopy(encTextByte,0,encTextAll,0,encTextByte.length);
System.arraycopy(encTagByte,0,encTextAll,encTextByte.length,encTagByte.length);
byte[] aad = headerSerializa.getBytes("US-ASCII");

byte[] s = decryptUseCekAndSalt(cek,Base64.decode(encIv),encTextAll, aad,split[0]);
System.out.println(new String(s,JoseConstants.UTF_8));
```

cekDecryption 方法（key的解密 非对称）
```
/**
 *  解密cek 使用公钥modulus 与 私钥 exponent RSA
 * @param encCek 加密后的key
 * @param radix 16 进制
 * @param publicModulus 公钥的模
 * @param privateExponent 私钥指数
 * @return
 * @throws Exception
 */
public static byte[] cekDecryption(byte[] encCek, int radix, String publicModulus, String privateExponent) throws Exception{
byte[] cek = null;
try {
    RSAPrivateKeySpec rsaPrivateKeySpec = new RSAPrivateKeySpec(new BigInteger(publicModulus,radix),new BigInteger(privateExponent,radix));
    KeyFactory keyFactory = KeyFactory.getInstance("RSA");
    PrivateKey privateKey = keyFactory.generatePrivate(rsaPrivateKeySpec);
    Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding", BouncyCastleProvider.PROVIDER_NAME);
    cipher.init(2,privateKey);
    cek = cipher.doFinal(encCek);
}catch (Exception e){
    e.printStackTrace();
}finally {
    if (encCek == null){
        throw new Exception("cekDecryption 解密失败");
    }
}
return cek;
}
```
decryptUseCekAndSalt （对称解密）
```
/**
 *  使用 cek / salt 解密 data 对称
 * @param cek
 * @param salt
 * @param data
 * @param aad 可以为null
 * @return
 */
public static byte[] decryptUseCekAndSalt(byte[] cek, byte[] salt, byte[] data, byte[] aad,String headerEnc) throws Exception {
    byte[] out = null;
    try {
        AlgorithmFactoryFactory factoryFactory = AlgorithmFactoryFactory.getInstance();
        AlgorithmFactory<ContentEncryptionAlgorithm> factory = factoryFactory.getJweContentEncryptionAlgorithmFactory();
        ContentEncryptionAlgorithm alg = factory.getAlgorithm("A128GCM");

        byte[] encTextByte = new byte[data.length -16];
        byte[] encTagByte = new byte[16];

        System.arraycopy(data,0,encTextByte,0,encTextByte.length);
        System.arraycopy(data,data.length-16,encTagByte,0,encTagByte.length);

        Headers headers = new Headers();
        headers.setFullHeaderAsJsonString(new String(Base64.decode(headerEnc),"UTF-8"));

        ContentEncryptionParts contentEncryptionParts = new ContentEncryptionParts(salt,encTextByte,encTagByte);
        // 解密
        out = alg.decrypt(contentEncryptionParts, aad, cek, headers, new ProviderContext());
        return out;
    }catch (Exception e){
        e.printStackTrace();
    }finally {
        if(out == null){
            throw new Exception("decryptUseCekAndSalt 解密失败");
        }
    }
    return null;
}

```

## 结束语
以上就是JWE的实现流程与方式，以及最终数据的组装拼接校验。
