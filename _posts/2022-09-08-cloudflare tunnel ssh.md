---
layout: post
title: cloudflare tunnel ssh
categories: [cloudflare, tunnel, ssh]
description: cloudflare tunnel ssh
keywords: cloudflare, tunnel, ssh
tags: cloudflare, tunnel, ssh
---

根据官方文档

    https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/use_cases/ssh/#1-connect-the-server-to-cloudflare
    https://developers.cloudflare.com/cloudflare-one/applications/non-http/

    https://dash.teams.cloudflare.com/a1b390d95b452b63912f898e4eb50b7f/home/quick-start    控制台


参考 
    https://johnrosen1.com/2022/04/19/cloudflare/

## 安装

    https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation/

## 登录

```shell
cloudflared tunnel login
```

有界面的会跳转到路蓝旗认证登录，无界面的命令行执行后会打印一个认证连接，复制粘贴到浏览器进行认证

## 隧道新建

```shell
cloudflared tunnel create <Tunnel-NAME>
# 会返回一个 UUID 字符串 Tunnel-UUID 对应的文件放在 ~/.cloudflared/<Tunnel-UUID>.json
```

## DNS设置

```shell
cloudflared tunnel route dns <Tunnel-NAME> <SUBDOMAIN>
# SUBDOMAIN 为二级域名的字符串，例如 eg.viakiba.cn 则 SUBDOMAIN 为 eg
```

## 新建配置

按照 Tunnel-UUID.json 存放位置，我们在同级路径下创建配置文件 config.yml .我们用来打通ssh通道，所以配置例子也是 ssh 的。

```yml
tunnel: e642dfff-794a-47c7-aa26-10ca93d7026a <Tunnel-UUID>
credentials-file: /Users/dd/.cloudflared/e642dfff-794a-47c7-aa26-10ca93d7026a.json  <Tunnel-UUID>.json
ingress:
  - hostname: <SUBDOMAIN>.viakiba.cn
    service: ssh://localhost:22
  - service: http_status:404
```

由于我们是打通 ssh 通道，我们需要修改 ~/.ssh/config 配置。

```shell
vim ~/.ssh/config
```
输入一下内容

```shell
Host <SUBDOMAIN>.viakiba.cn
ProxyCommand /usr/local/bin/cloudflared access ssh --hostname %h
# /usr/local/bin/cloudflared 按照自己存放的路径进行设置
```

## 运行 cloudflared tunnel

```shell
cloudflared tunnel --config ~/.cloudflared/config.yml run
```

## 测试

```shell
# 其他终端
ssh root@<SUBDOMAIN>.viakiba.cn
# 运行该命令时，cloudflared将启动一个浏览器窗口，提示您在从终端建立连接之前向您的身份提供者进行身份验证。
```

## 补充

cloudflare 提供了 浏览器渲染的终端方式，所以我们可以使用浏览器访问 https://<SUBDOMAIN>.viakiba.cn ,验证后就能看到呈现的终端。

使用此方式，我们需要进行如下操作。

- 进入 https://dash.teams.cloudflare.com/ 控制后台。
- 点击 access - Applications  , 点击 Add an application .
- 可以选择 Self-hosted ,进入 Configure app 
  - Application name 设置为 <SUBDOMAIN>
  - Application domain Subdomain 设置为 <SUBDOMAIN>
  - Domain 设置为 viakiba.cn
- 点击 next 进入 Add policies
  - Policy name 设置为 <SUBDOMAIN>
  - Session duration 可以短一些，15分钟看需求。
  - Configure rules 按需选择，我选择的 Login Method ,pin码。
- 点击next进入 SetUp
  - Additional settings 选择 ssh.
- 点击 Add Application 即可完成添加。
  
此时，就可以按照前面所说的使用浏览器渲染的终端进行访问。