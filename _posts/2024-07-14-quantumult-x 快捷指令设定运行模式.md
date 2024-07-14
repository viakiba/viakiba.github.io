---
layout: post
title: QuantumultX 快捷指令设定运行模式
categories: [QuantumultX]
description: QuantumultX
keywords: QuantumultX, QX, HTTP Backend
tags: qQuantumultX, QX, HTTP Backend
---


记录一下如何用快捷指令控制 QuantumultX (QX) 的运行模式，使用了 QX 的 HTTP Backend 功能，所以需要开启 HTTP Backend 功能，并且运行 QX.
> 把如下的 js 保存文件到手机中。

```js

// 这是一个简单的 HTTP Backend 代码例子 https://github.com/crossutility/Quantumult-X/blob/master/sample-backend.js
// 这是控制 QX 运行模式的 代码例子  https://github.com/crossutility/Quantumult-X/blob/master/sample-configuration-api.js

// The availabel variables: $request.url, $request.path, $request.headers, $request.body, $prefs, $task, $notify(title, subtitle, message), console.log(message), $done(response)

const myStatus = "HTTP/1.1 200 OK";
const myHeaders = {"Connection": "Close"};


const url = $request.url
const path = $request.path

// 获取当前运行模式
if (path.includes("getmode")){
    const message = {
        action: "get_running_mode"
    };
    $configuration.sendMessage(message).then(resolve => {
        if (resolve.error) {
            console.log(resolve.error);
        }
        if (resolve.ret) {
            let output=JSON.stringify(resolve.ret);
            const myResponse = {
                status: myStatus,
                headers: myHeaders,
                body: output
            };
            $done(myResponse);
        }
    }, reject => {
        const myResponse = {
            status: myStatus,
            headers: myHeaders,
            body: JSON.stringify("notfindstate")
        };
        $done(myResponse);
    });
}else if (path.includes("setmodeproxy")){
    // 设置当前运行模式
    // all_proxy, all_direct, filter
    const dict = { "running_mode": "all_proxy" };
    const messagesetmode = {
        action: "set_running_mode",
        content: dict
    };

    $configuration.sendMessage(messagesetmode).then(resolve => {
        if (resolve.ret) {
            let output=JSON.stringify(resolve.ret); 
            const myResponse = {
                status: myStatus,
                headers: myHeaders,
                body: output
            };
            $done(myResponse);
        }
        
    }, reject => {
        const myResponse = {
            status: myStatus,
            headers: myHeaders,
            body: JSON.stringify("notsetstatesetmodeproxy")
        };
        $done(myResponse);
    });
}else if (path.includes("setmodedirect")){
    // 设置当前运行模式
    // all_proxy, all_direct, filter
    const dict = { "running_mode": "all_direct" };
    const messagesetmode = {
        action: "set_running_mode",
        content: dict
    };

    $configuration.sendMessage(messagesetmode).then(resolve => {
        if (resolve.ret) {
            let output=JSON.stringify(resolve.ret); 
            const myResponse = {
                status: myStatus,
                headers: myHeaders,
                body: output
            };
            $done(myResponse);
        }
        
    }, reject => {
        const myResponse = {
            status: myStatus,
            headers: myHeaders,
            body: JSON.stringify("notsetstateall_direct")
        };
        $done(myResponse);
    });
}else if (path.includes("setmodefilter")){
    // 设置当前运行模式
    // all_proxy, all_direct, filter
    const dict = { "running_mode": "filter" };
    const messagesetmode = {
        action: "set_running_mode",
        content: dict
    };

    $configuration.sendMessage(messagesetmode).then(resolve => {
        if (resolve.ret) {
            let output=JSON.stringify(resolve.ret); 
            const myResponse = {
                status: myStatus,
                headers: myHeaders,
                body: output
            };
            $done(myResponse);
        }
        
    }, reject => {
        const myResponse = {
            status: myStatus,
            headers: myHeaders,
            body: JSON.stringify("notsetstatefilter")
        };
        $done(myResponse);
    });
}else{
    const myResponse = {
        status: myStatus,
        headers: myHeaders,
        body: JSON.stringify("notfindpath")
    };
    $done(myResponse);
}

```

在QX的设置中找到 工具&分享里的 HTTP Backend ，点击后新建一个backend.如下图：

![ss](/images/post/2024/QX.jpg)

注意截图里的 处理请求的路径 */change/** 与这个一样可以直接使用下面的快捷指令与 http 请求例子。脚本路径选择刚刚保存好的脚本。

```http
获取当前运行模式 http://127.0.0.1:9999/change/getmode
设置当前运行模式为全部代理   http://127.0.0.1:9999/change/setmodeproxy
设置当前运行模式为全部直连   http://127.0.0.1:9999/change/setmodedirect
设置当前运行模式为规则代理   http://127.0.0.1:9999/change/setmodefilter

以上链接 在打开 QX 运行 及其 http-Backend 开关后，浏览器访问以上链接即可更改 QX 的运行模式。

也可以使用编辑好的快捷指令

https://www.icloud.com/shortcuts/e77bcff267894ae29c302234a535cec4
```

打开 QX-backend

![ss](/images/post/2024/QX-back.png)