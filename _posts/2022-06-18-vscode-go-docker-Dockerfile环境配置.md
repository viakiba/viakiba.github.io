---
layout: post
title: vscode-go-docker-Dockerfile环境配置
categories: [vscode, go, docker]
description: vscode-go-docker环境配置
keywords: vscode, go, docker, 环境配置
---

# [源代码](https://github.com/viakiba/viakiba/tree/master/single-go-docker-0)

# 单体应用

    单体应用是指一个程序只有一个入口点，可以被单独运行的程序。在这种场景下，我们构建起docker容器，以 vscode-go-docker 的方式运行单体应用。这样一次实现配置文件，我们就可以在多处依赖配置文件借助vscode快速拉起一套开发环境。整个体验与在宿主机中体验完全一致。

## 依赖

1. vscode 最新版
   
   1. 依赖插件: https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack
   
2. docker 最新版

## 配置文件

### Dockerfile

dockerfile 我们这里是比较简单的，直接使用golang指定版本的基础镜像即可，安装必要的依赖，如 git 等。亦可以看自己的需求增加一些额外的命令。
```Dockerfile
FROM golang:1.18.3-bullseye
# Configure to reduce warnings and limitations as instruction from official VSCode Remote-Containers.
# See https://code.visualstudio.com/docs/remote/containers-advanced#_reducing-dockerfile-build-warnings.
RUN apt-get update 
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get -y install git iproute2 procps lsb-release
```

### .devcontainer.json

在根目录创建 .devcontainer.json 文件 （注意最前面有一个 .）。

```jsonc
{
    "dockerFile": "Dockerfile", // 指定 Dockerfile 名称
    "appPort": [
        "8000:8001" // 将 容器的8001 端口映射到 宿主机的 8000 端口
    ],
    "extensions": [ // 安装 vscode-go-docker 的扩展 这里声明的是扩展的 扩展id
        "ms-vscode.go",
        "GitHub.copilot",
        "golang.go",
        "mrcrowl.hg",
        "oderwat.indent-rainbow",
        "VisualStudioExptTeam.vscodeintellicode",
        "mohsen1.prettify-json",
        "johnstoncode.svn-scm",
        "zxh404.vscode-proto3"
    ]
}
//这里还有很多参数配置 https://code.visualstudio.com/docs/remote/devcontainerjson-reference ，可以根据自身情况进行配置。例如 环境变量。
// 文件夹会自动从容器映射到宿主机的目录下。
```

![x](/images/post/2022/扩展id.png)

### vscode

此时，使用vscode 打开此文件夹，会识别到 .devcontainer.json 文件，并且提示会自动拉起一个容器。点击 reopen in container ,vscode 重新加载后就处于容器内部了。(注意docker需要在宿主机上安装并运行)

![](/images/post/2022/single-go-docker-0-0.png)

### 补充

```shell
go install github.com/go-delve/delve/cmd/dlv@latest 
echo "command_init_env.sh: go install github.com/go-delve/delve/cmd/dlv@latest"
go install -v golang.org/x/tools/gopls@latest 
echo "command_init_env.sh: go install golang.org/x/tools/gopls@latest"
go get golang.org/x/tools/gopls@latest 
echo "command_init_env.sh: go get golang.org/x/tools/gopls@latest"
go install -v github.com/stamblerre/gocode@latest 
echo "command_init_env.sh: go install github.com/stamblerre/gocode@latest"
go install -v golang.org/x/tools/cmd/goimports@latest   
echo "command_init_env.sh: go install golang.org/x/tools/cmd/goimports@latest"
go install -v github.com/ramya-rao-a/go-outline@latest 
echo "command_init_env.sh: go install github.com/ramya-rao-a/go-outline@latest"
go install -v github.com/rogpeppe/godef@latest
echo "command_init_env.sh: go install github.com/rogpeppe/godef@latest"
go install -v honnef.co/go/tools/cmd/staticcheck@latest
echo "command_init_env.sh: go install honnef.co/go/tools/cmd/staticcheck@latest"
```

由于 dockerfile 中未安装 go 相关的 注入 dlv(debug), staticcheck (检查) 等工具，可以使用以下命令在容器中进行执行安装。安装后如果未识别，可以重新连入容器即可。也可重复执行一下命令。

```shell
go install github.com/go-delve/delve/cmd/dlv@latest 
echo "command_init_env.sh: go install github.com/go-delve/delve/cmd/dlv@latest"
go install -v golang.org/x/tools/gopls@latest 
echo "command_init_env.sh: go install golang.org/x/tools/gopls@latest"
go get golang.org/x/tools/gopls@latest 
echo "command_init_env.sh: go get golang.org/x/tools/gopls@latest"
go install -v github.com/stamblerre/gocode@latest 
echo "command_init_env.sh: go install github.com/stamblerre/gocode@latest"
go install -v golang.org/x/tools/cmd/goimports@latest   
echo "command_init_env.sh: go install golang.org/x/tools/cmd/goimports@latest"
go install -v github.com/ramya-rao-a/go-outline@latest 
echo "command_init_env.sh: go install github.com/ramya-rao-a/go-outline@latest"
go install -v github.com/rogpeppe/godef@latest
echo "command_init_env.sh: go install github.com/rogpeppe/godef@latest"
go install -v honnef.co/go/tools/cmd/staticcheck@latest
echo "command_init_env.sh: go install honnef.co/go/tools/cmd/staticcheck@latest"
```

这些命令也可以放到 dockerfile 中，然后在容器中使用。

### tips

建议 容器内存给大一些，最少2g。不然可能会遇到 vscode 频繁重新连接以及卡顿的问题。

![](/images/post/2022/WX20220618-192019.png)

### 安装工具（容器）
![](/images/post/2022/WX20220618-192235.png)

## 项目代码
### 初始化一个go项目（也可以是现有的）
```shell
root@f16110fb8999:/workspaces/single-go-docker-0# go mod init single.go.demo/demo
go: creating new go.mod: module single.go.demo/demo
root@f16110fb8999:/workspaces/single-go-docker-0# go mod tidy
go: finding module for package github.com/gin-gonic/gin
go: downloading github.com/gin-gonic/gin v1.8.1
go: found github.com/gin-gonic/gin in github.com/gin-gonic/gin v1.8.1
go: downloading github.com/gin-contrib/sse v0.1.0
go: downloading golang.org/x/net v0.0.0-20210226172049-e18ecbb05110
go: downloading github.com/mattn/go-isatty v0.0.14
go: downloading github.com/stretchr/testify v1.7.1
go: downloading google.golang.org/protobuf v1.28.0
go: downloading github.com/goccy/go-json v0.9.7
go: downloading github.com/json-iterator/go v1.1.12
go: downloading github.com/pelletier/go-toml/v2 v2.0.1
go: downloading github.com/ugorji/go/codec v1.2.7
go: downloading github.com/go-playground/validator/v10 v10.10.0
go: downloading golang.org/x/sys v0.0.0-20210806184541-e5e7981a1069
go: downloading github.com/davecgh/go-spew v1.1.1
go: downloading github.com/pmezard/go-difflib v1.0.0
go: downloading gopkg.in/yaml.v3 v3.0.0-20210107192922-496545a6307b
go: downloading github.com/modern-go/concurrent v0.0.0-20180228061459-e0a39a4cb421
go: downloading github.com/modern-go/reflect2 v1.0.2
go: downloading golang.org/x/text v0.3.6
go: downloading github.com/go-playground/universal-translator v0.18.0
go: downloading github.com/leodido/go-urn v1.2.1
go: downloading golang.org/x/crypto v0.0.0-20210711020723-a769d52b0f97
go: downloading github.com/go-playground/locales v0.14.0
go: downloading github.com/go-playground/assert/v2 v2.0.1
go: downloading github.com/google/go-cmp v0.5.5
go: downloading gopkg.in/check.v1 v1.0.0-20201130134442-10cb98267c6c
go: downloading github.com/kr/pretty v0.3.0
go: downloading golang.org/x/xerrors v0.0.0-20191204190536-9bdfabe68543
go: downloading github.com/rogpeppe/go-internal v1.8.0
go: downloading github.com/kr/text v0.2.0
root@f16110fb8999:/workspaces/single-go-docker-0# 
```
![](/images/post/2022/WX20220618-192747.png)

### 运行

![](/images/post/2022/WechatIMG1.png)

根据 .devcontainer.json 文件的端口映射配置，代码监听端口为 8001。宿主机浏览器访问 http://localhost:8000/ping 即可。

### 连接到redis

创建一个redis服务器，无密码。
```shell
docker run -itd --name redis-test -p 6380:6379 redis
# 容器 6379 映射到 宿主机 6380 上
```

    改造一下刚刚的 main.go 里的程序，增加两个 url ，一个是 [setRedisKey](http://0.0.0.0:8000/setRedisKey) 一个是 [getRedisKey](http://0.0.0.0:8000/getRedisKey) .增加redis依赖。

    go-redis 客户端 这里使用的是 github.com/go-redis/redis 客户端。与 redis server 有版本对应关系。我的redis server 是 7，所以 依赖使用 go get github.com/go-redis/redis/v9 。

#### 直连宿主机

```
    // host.docker.internal 这个 地址 是 docker的一个特性，可以访问宿主机端口。
    rdb := redis.NewClient(&redis.Options{
		Addr:     "host.docker.internal:6380",
		Password: "", // no password set
		DB:       0,  // use default DB
	})
	sc := rdb.Ping(context.Background())
	print(sc)
	r := gin.New()
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})

	r.GET("/setRedisKey", func(c *gin.Context) {
		rdb.Set(context.Background(), "key1", "value111", 10*time.Second)
		c.JSON(200, gin.H{
			"message": "pong1",
		})
	})

	r.GET("/getRedisKey", func(c *gin.Context) {
		value := rdb.Get(context.Background(), "key1")
		println(value.String())
		c.JSON(200, gin.H{
			"message": "pong2",
		})
	})
	r.Run(":8001") // listen and serve on
```

#### 连接redis容器（非直连宿主机）

```go
	// 连接redis容器（非直连宿主机）
	// shell下执行    docker network inspect bridge
	// Containers 标签下  name 为 redis-test 对应的 ip 地址 172 开头
	rdb := redis.NewClient(&redis.Options{
		Addr:     "172.17.0.3:6379",
		Password: "", // no password set
		DB:       0,  // use default DB
	})
```


这种方式仍然不是使用docker环境的最佳方式，有redis此类服务多个依赖更好的方式是使用 docker-compose。下一篇文章进行详细介绍与实践。

#### 连接redis容器（docker 虚拟网络）

```shell
# 创建一个虚拟网络 名字为 redis-test-net
docker network create test-redis-net
```
启动一个redis容器，并且指定虚拟网络。
```shell
docker run -d --name redis-test-net --network test-redis-net --network-alias redisnet redis:latest
# --network-alias redis 网络别名
# -p 6381:6379 无需指定端口
# --network test-redis-net 属于 redis-test-net 虚拟网络
```

	改造一下 .devcontainer.json ，增加如下 [runArgs](https://code.visualstudio.com/docs/remote/devcontainerjson-reference#_image-or-dockerfile-specific-properties) 参数

```jsonc
{
    "dockerFile": "Dockerfile",
    "appPort": [
        "8000:8001"
    ],
    // 容器监听端口 8001 映射到本地端口 8000
    "extensions": [
        "ms-vscode.go",
        "GitHub.copilot",
        "golang.go",
        "mrcrowl.hg",
        "oderwat.indent-rainbow",
        "VisualStudioExptTeam.vscodeintellicode",
        "mohsen1.prettify-json",
        "johnstoncode.svn-scm",
        "zxh404.vscode-proto3"
    ],
    "runArgs":[
        "--network=test-redis-net"
    ]
}
// https://code.visualstudio.com/docs/remote/devcontainerjson-reference
```
注意： 此时由于修改了 .devcontainer.json ，所以需要重新 build 一下容器。需要重新执行 sh command_init_env.sh 与 go mod tidy 。

修改代码

```go
// redisnet
// docker run -d --name redis-test-net --network test-redis-net --network-alias redisnet redis:latest
// --network-alias 声明的别名
rdb := redis.NewClient(&redis.Options{
	Addr:     "redisnet:6379",
	Password: "", // no password set
	DB:       0,  // use default DB
})
```