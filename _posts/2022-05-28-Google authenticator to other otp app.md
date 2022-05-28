---
layout: post
title: Google authenticator to other otp app
categories: [App, 工具]
description: authenticator
keywords: App，工具
---

## 背景

    最近想从 Google authenticator (没有云备份) 这个 otp 应用迁移到其他 app上，但是这个app的导出格式私有的。比如 authy , 微软 authenticator 等都不识别，所以我们需要从 Google authenticator这个二维码获取对应的 totpSecret .然后以输入的方式添加到其他app上。

推荐一个github仓库就可以干这个事情 [仓库连接](https://github.com/krissrex/google-authenticator-exporter)

```shell
huangpeng@huangpengdeMac-mini google-authenticator-exporter % npm run start

> google-auth-exporter@1.0.0 start
> node ./src/index.js

Enter the URI from Google Authenticator QR code.
The URI looks like otpauth-migration://offline?data=... 

You can get it by exporting from Google Authenticator app, then scanning the QR with
e.g. https://play.google.com/store/apps/details?id=com.google.zxing.client.android
and copying the text to your pc, e.g. with Google Keep ( https://keep.google.com/ )
By using online QR decoders or untrusted ways of transferring the URI text,
you risk someone storing the QR code or URI text and stealing your 2FA codes!
Remember that the data contains the website, your email and the 2FA code!
prompt: totpUri:  otpauth-migration://offline?data=Ci0KCkSWojbxxxxxxxxxxxxaXRIdWIgASgBMAIQARxxxA%xx
prompt: saveToFile:  n
prompt: filename:  f
f
false
Not saving. Here is the data:
[
  {
    secret: 'RJaiNuYUCLasLA==',
    name: 'xxx@xx.com',
    issuer: 'GitHub',
    algorithm: 'ALGO_SHA1',
    digits: 1,
    type: 'OTP_TOTP',
    totpSecret: 'ISLKENXGCQELNLBM'    // 这个就是其他app需要的 secret
  }
]
What you want to use as secret key in other password managers is 'totpSecret', not 'secret'!
```