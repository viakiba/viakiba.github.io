---
layout: post
title: vscode-go-docker-compose环境配置
categories: [vscode, go, docker, compose]
description: vscode-go-docker环境配置
keywords: vscode, go, docker, compose, 环境配置
---

**以下文章未设置 go代理 ，即[链接](https://goproxy.io/zh/)**

```shell
go env -w GO111MODULE=on
go env -w GOPROXY=https://proxy.golang.com.cn,direct
```
或
```
# 配置 GOPROXY 环境变量
export GOPROXY=https://proxy.golang.com.cn,direct
# 还可以设置不走 proxy 的私有仓库或组，多个用逗号相隔（可选）
export GOPRIVATE=git.mycompany.com,github.com/my/private
```

**[docker compose 入门](https://docker.easydoc.net/doc/81170005/cCewZWoN/IJJcUk5J)**

**继续[上一篇文章](https://blog.viakiba.cn/2022/06/18/vscode-go-docker-Dockerfile%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE/)提到的 docker-compose, 这一次我们根据配置文件直接拉起配置环境以及所需要的redis环境。**

**以下构建的环境，只要不重新构建容器，都是一次设置，后续使用都是会保存数据的。**

**[源代码](https://github.com/viakiba/viakiba/tree/master/go-docker-compose-demo)**

**[代码参考](https://github.com/Microsoft/vscode-dev-containers/tree/main/containers/go-postgres)**

**[配置参考](https://code.visualstudio.com/docs/remote/devcontainerjson-reference)**

# 配置文件

层级目录如下图：

![](/images/post/2022/WX20220619-103930.png)

与[上篇文章](https://blog.viakiba.cn/2022/06/18/vscode-go-docker-Dockerfile%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE/)相比，我们创建了 .devcontainer 文件夹，把 .devcontainer.json 文件移动到了 .devcontainer 文件夹下，并且把 .devcontainer.json 文件名称换成了 devcontainer.json，同样的 Dockerfile 文件 也放到了 .devcontainer 文件夹下。新增了 .env 文件，用于配置环境变量 以及 新增了 docker-compose.yml 文件，用于配置 docker-compose 环境。

## Dockerfile

与[上篇文章](https://blog.viakiba.cn/2022/06/18/vscode-go-docker-Dockerfile%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE/)完全一致。

```Dockerfile
FROM golang:1.18.3-bullseye
# Configure to reduce warnings and limitations as instruction from official VSCode Remote-Containers.
# See https://code.visualstudio.com/docs/remote/containers-advanced#_reducing-dockerfile-build-warnings.
RUN apt-get update 
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get -y install git iproute2 procps lsb-release
```

## .env

此文件为新增的，我只是象征性的增加了一个测试环境变量

```env
testenv=qqq
```
## docker-compose.yml

[五分钟入门](https://docker.easydoc.net/doc/81170005/cCewZWoN/IJJcUk5J)

```yaml
version: '3.8'

volumes:
  redis-data:
  #   services.redis.volumes下有引用 redis-data:/data

services:
  app:
    build: 
      context: .
    #   引用 Dockerfile 文件 (同级目录)
      dockerfile: Dockerfile
    env_file:
        # 引用环境变量配置文件
        - .env
    volumes:
      - ..:/workspace:cached
    command: sleep infinity
    network_mode: service:redis
    # forward_port:
    #   80:80
    # 端口映射
  redis:
    image: redis:7.0.2
    restart: unless-stopped
    volumes:
      - redis-data:/data
    environment:
      - TZ=Asia/Shanghai
    env_file:
      - .env
    # forwardPorts:
    #   - "6379:6379"
    # 端口映射
```

## devcontainer.json

```jsonc
// https://code.visualstudio.com/docs/remote/devcontainerjson-reference#_general-devcontainerjson-properties
// https://code.visualstudio.com/docs/remote/devcontainerjson-reference#_docker-compose-specific-properties
// https://code.visualstudio.com/docs/remote/devcontainerjson-reference
{	
	"name": "Go & Redis",
	"dockerComposeFile": "docker-compose.yml",
	"service": "app",
	"workspaceFolder": "/workspace",
	// Indicates whether VS Code and other devcontainer.json supporting tools should stop the containers when the related tool window is closed / shut down. Values are none, stopContainer (default for image or Dockerfile), and stopCompose (default for Docker Compose).
	//"shutdownAction": "stopCompose",
	//配置工具特定的属性。
	//在创建容器时添加要安装的扩展的 ID。
	"extensions": [
		"golang.Go",
		"ms-vscode.go",
		"GitHub.copilot",
		"mrcrowl.hg",
		"oderwat.indent-rainbow",
		"VisualStudioExptTeam.vscodeintellicode",
		"mohsen1.prettify-json",
		"johnstoncode.svn-scm",
		"zxh404.vscode-proto3"
	],
	"settings": { 
		"go.toolsManagement.checkForUpdates": "local",
		"go.useLanguageServer": true,
		"go.gopath": "/go",
		"go.goroot": "/usr/local/go"
	},
	"customizations": {
		"vscode": {
			// 在容器创建时设置 *default*容器特定的 settings.json 值
			"settings": { 
				// "go.toolsManagement.checkForUpdates": "local",
				// "go.useLanguageServer": true,
				// "go.gopath": "/go",
				// "go.goroot": "/usr/local/go"
			},
			"extensions": [
				// "golang.Go",
				// "ms-vscode.go",
				// "GitHub.copilot",
				// "mrcrowl.hg",
				// "oderwat.indent-rainbow",
				// "VisualStudioExptTeam.vscodeintellicode",
				// "mohsen1.prettify-json",
				// "johnstoncode.svn-scm",
				// "zxh404.vscode-proto3"
			]
		}
	}
	// 使用 "forwardPorts" 在宿主机可以访问 端口映射
	// "forwardPorts": [5432],
	// 创建容器后使用 "postCreateCommand" 运行命令。
	// "postCreateCommand": "go version",
	// 远程访问默认以 root 用户访问，打开注释以 vscode 用户访问。https://aka.ms/vscode-remote/containers/non-root.
	// "remoteUser": "vscode"
}
```

其余文件，与[上篇文章](https://blog.viakiba.cn/2022/06/18/vscode-go-docker-Dockerfile%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE/) 相同。

![](/images/post/2022/WX20220619-105624.png)

当我们使用 vscode 的 reopen in container 时，容器会自动拉起。第一次时我们执行 sh command_init_env.sh 初始化环境，go mod tidy 更新依赖。然后就可以在debug里执行 cmd/main.go 了。

注意修改 cmd/main.go 里的redis连接代码：

```go
rdb := redis.NewClient(&redis.Options{
    Addr:     "redis:6379",
    Password: "", // no password set
    DB:       0,  // use default DB
})
```

Addr 的 ip 是 redis的原因是，我们在 docker-compose.yml 里面，使用 network_mode: "service:[service name]" 方式，service name 是 ridis .所以这里写的是 service name.

## docker desktop 截图

![](/images/post/2022/WechatIMG2.png)