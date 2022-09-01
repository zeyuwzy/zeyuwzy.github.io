---
title: Docker - 02Client
date: 2022-08-30 09:00:00
tags:
    - Docker
---

## 写在前面
自从`20.10`版本,原来`Docker-CE`的仓库就不再使用，而是分成了[`Docker CLI`](https://github.com/docker/cli)和[Docker Engine](https://github.com/moby/moby),后续涉及源码的地方都将基于`20.10`展开

## Docker Client
```
​​docker [OPTIONS] COMMAND [arg...]​​
```

`docker client`入口函数在`cmd/docker/docker.go`

### 01 初始化DockerCli

调用`cli/command/cli.go/NewDockerCli`实例化一个cli,这里会去获取环境变量、设置默认的`store.Config`、配置默认的标准输入输出及错误输出

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
4. 解析传入的`Options` `cli/cobra.go/HandleGlobalFlags`,将os.Args的`Options`切分后面就是`args`
5. 初始化`Options` `cli/cobra.go/Initialize`, 初始化`APIClient`(用于和daemon通信)
5. 解析`os.Args`后面的args,匹配相关命令
6. 执行匹配到的命令