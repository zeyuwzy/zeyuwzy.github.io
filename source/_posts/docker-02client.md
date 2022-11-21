---
title: Docker - 02Client
date: 2022-08-30 09:00:00
tags:
    - Docker
---

## 写在前面
自从`20.10`版本,原来`Docker-CE`的仓库就不再使用，而是分成了[`Docker CLI`](https://github.com/docker/cli)和[Docker Engine](https://github.com/moby/moby),后续涉及源码的地方都将基于`v20.10.12`展开

## Docker Client
```
​​docker [OPTIONS] COMMAND [arg...]​​
```

`docker client`入口函数在`cmd/docker/docker.go`

### 01 初始化DockerCli

调用`cli/command/cli.go/NewDockerCli`实例化一个`cli`,这里会去获取环境变量、设置默认的`store.Config`、配置默认的标准输入输出及错误输出

`DockerCli`将作为客户端的上下文存在于整个生命周期

```go
type DockerCli struct {
	configFile         *configfile.ConfigFile
	in                 *streams.In
	out                *streams.Out
	err                io.Writer
	client             client.APIClient
	serverInfo         ServerInfo
	clientInfo         *ClientInfo
	contentTrust       bool
	contextStore       store.Store
	currentContext     string
	dockerEndpoint     docker.Endpoint
	contextStoreConfig store.Config
}
```

### 02 runDocker
1. 初始化命令模版: 在`newDockerCommand`中初始化命令行模版,命令模版:`vendor/github.com/spf13/cobra/command.go/command`
2. 初始化`Flags`并关联`Options`: `cli/flags/common.go/InstallFlags` 
    ```go
    type CommonOptions struct {
        Debug      bool
        Hosts      []string
        LogLevel   string
        TLS        bool
        TLSVerify  bool
        TLSOptions *tlsconfig.Options
        Context    string
    }
    ```
3. 初始化命令: `cli/command/commands/commands.go/AddCommands`,`cli/command/container`下包含容器相关的命令,`cli/command/image`下包含镜像相关的命令,以此类推
4. 解析传入的`Options` `cli/cobra.go/HandleGlobalFlags`,将`os.Args`的`Options`切分后面就是`args`
5. 初始化`Options` `cli/cobra.go/Initialize`, 初始化`APIClient`(用于和`daemon`通信)
5. 解析`os.Args`后面的`args`,匹配相关命令
6. 执行匹配到的命令

## 源码编译cli
`cli`是在`docker`容器中编译的,但是通过`dockerfile`构建容器时可能会遇到网络问题导致部分依赖拉取失败

1. 修改`dockerfiles/Dockerfile.binary-native`,换源

    ```dockerfile
    #...
    FROM    golang:${GO_VERSION}-alpine
    #换源
    RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

    RUN     apk add -U git bash coreutils gcc musl-dev
    #...
    ```

2. 修改`dockerfiles/Dockerfile.dev`,添加代理, `GOPROXY=https://goproxy.cn,direct`
    ```dockerfile
    # ...
    FROM golang AS esc
    ARG ESC_VERSION=v0.2.0
    RUN --mount=type=cache,target=/root/.cache/go-build \
        --mount=type=cache,target=/go/pkg/mod \
        --mount=type=tmpfs,target=/go/src/ \
        GO111MODULE=on GOPROXY=https://goproxy.cn,direct go install github.com/mjibson/esc@${ESC_VERSION}

    FROM golang AS gotestsum
    ARG GOTESTSUM_VERSION=v0.4.0
    RUN --mount=type=cache,target=/root/.cache/go-build \
        --mount=type=cache,target=/go/pkg/mod \
        --mount=type=tmpfs,target=/go/src/ \
        GO111MODULE=on GOPROXY=https://goproxy.cn,direct go install gotest.tools/gotestsum@${GOTESTSUM_VERSION}

    FROM golang AS vndr
    ARG VNDR_VERSION=v0.1.2
    RUN --mount=type=cache,target=/root/.cache/go-build \
        --mount=type=cache,target=/go/pkg/mod \
        --mount=type=tmpfs,target=/go/src/ \
        GO111MODULE=on GOPROXY=https://goproxy.cn,direct go install github.com/LK4D4/vndr@${VNDR_VERSION}
    #...
    ```

3. `sudo make -f docker.Makefile binary`, 最后二进制文件在工作目录下的`build`