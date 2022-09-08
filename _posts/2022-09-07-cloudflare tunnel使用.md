
---
layout: post
title: 斐讯N1小钢炮配置记录
categories: [斐讯N1, 下载器, 旁路由]
description: 斐讯N1小钢炮配置记录
keywords: 斐讯N1, 下载器, 旁路由
tags: 斐讯N1, 下载器, 旁路由
---





> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [johnrosen1.com](https://johnrosen1.com/2022/04/19/cloudflare/)

> Cloudflare 隧道内网穿透搭建记录由于国内运营商不让用 80 端口，备案又麻烦，因此我想出了这招。

由于国内运营商不让用 80 端口，备案又麻烦，因此我想出了这招。

[](#优缺点 "优缺点")优缺点
-----------------

1.  免费且不需要服务器
2.  暂时不支持 UDP 协议

[](#前提条件 "前提条件")前提条件
--------------------

1.  一个托管于 Cloudflare 的域名，相关教程看这里[创建 Cloudflare 帐户并添加网站](https://support.cloudflare.com/hc/zh-cn/articles/201720164-%E5%88%9B%E5%BB%BA-Cloudflare-%E5%B8%90%E6%88%B7%E5%B9%B6%E6%B7%BB%E5%8A%A0%E7%BD%91%E7%AB%99)
2.  一台本地 Linux Web 机器，即内网穿透的对象
3.  正常网络连接

[](#安装-Cloudflared "安装 Cloudflared")安装 Cloudflared
--------------------------------------------------

```
curl -LO https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
dpkg -i cloudflared-linux-amd64.deb
rm cloudflared-linux-amd64.deb
```

[](#登录-Cloudflared "登录 Cloudflared")登录 Cloudflared
--------------------------------------------------

```
cloudflared tunnel login
```

这时会弹出来一个 URL，用浏览器打开，登录认证，然后选择你想用来做内网穿透的域名即可。

成功后会生成证书，放置于`~/cloudflared/cert.pem`中。

[](#新建隧道 "新建隧道")新建隧道
--------------------

名字可以随意起

```
cloudflared tunnel create <Tunnel-NAME>
```

成功后会提示，相关凭证已放置于`~/.cloudflared/<Tunnel-UUID>.json`中。

[](#新建-Tunnel-对应的-DNS-记录 "新建 Tunnel 对应的 DNS 记录")新建 Tunnel 对应的 DNS 记录
--------------------------------------------------------------------

`<SUBDOMAIN>`填你想用来做内网穿透的域名，可以为二级域名（example.com）或三级域名（[www.example.com）等。](http://www.example.xn--com%29-1v3p./)

```
cloudflared tunnel route dns <Tunnel-NAME> <SUBDOMAIN>
```

成功后会创建 CNAME 记录将域名指向隧道。

[](#新建配置文件 "新建配置文件")新建配置文件
--------------------------

```
nano ~/.cloudflared/config.yml
```

写入以下配置

```
tunnel: <Tunnel-UUID>
credentials-file: /root/.cloudflared/<Tunnel-UUID>.json
protocol: h2mux
originRequest:
  connectTimeout: 30s
  noTLSVerify: false
ingress:
  - hostname: <SUBDOMAIN>
    service: http://localhost:80
  - service: http_status:404
```

> 2022.4.20 更新：经测试，`http2`与`h2mux`均可正常建立连接，只有`quic`无法建立连接。

由于国内环境关系，无法使用默认的`quic`建立隧道，因此需指定`http2`,`http://localhost:80`为本地服务的地址。

    也可以不加入系统服务如下图指令，核心指令是

```shell
/usr/bin/cloudflared --loglevel debug --transport-loglevel warn --config /root/.cloudflared/config.yml tunnel run <Tunnel-NAME>
```


[](#启动-Cloudflared "启动 Cloudflared")启动 Cloudflared
--------------------------------------------------

修改 systemd 文件

```
nano /etc/systemd/system/cloudflared.service
```

写入以下内容

```
[Unit]
Description=cloudflared
After=network.target

[Service]
TimeoutStartSec=0
Type=notify
ExecStart=/usr/bin/cloudflared --loglevel debug --transport-loglevel warn --config /root/.cloudflared/config.yml tunnel run <Tunnel-NAME>
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

启动 Cloudflared

```
systemctl enable cloudflared --now
```

[](#测试穿透是否成功 "测试穿透是否成功")测试穿透是否成功
--------------------------------

等待一两分钟，然后尝试访问`https://<SUBDOMAIN>`，如可正常显示则成功。

```
cloudflared tunnel list
```

> Debug 指令：`systemctl status cloudflared` `journalctl -a -u cloudflared (-r / -f)`

[](#透明代理 "透明代理")透明代理
--------------------

由于 Cloudflare 走 V2ray 的 sniffing 或者 fakedns 都会出错，因此需专门写一条 iptables 规则开机启动。

```
crontab -e
```

设置规则

```
@reboot sleep 30s && iptables -t nat -I OUTPUT -p tcp --dport 7844 -j RETURN
@reboot sleep 30s && iptables -t nat -I OUTPUT -p udp --dport 7844 -j RETURN
```

[](#总结 "总结")总结
--------------

这玩意不比什么花生壳，frp 香多了，免费又不需要服务器，哈哈哈哈哈哈哈哈哈。

[](#参考资料 "参考资料")参考资料
--------------------

1.  [Cloudflare Docs](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/)
2.  [Many services, one cloudflared](https://blog.cloudflare.com/many-services-one-cloudflared/)