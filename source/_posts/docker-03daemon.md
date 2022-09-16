---
title: Docker - 03Daemon
date: 2022-09-01 09:00:00
tags:
    - Docker
---

## Docker Daemon
```
dockerd [OPTIONS]
```

服务端的入口函数在`cmd/dockerd/docker.go`

dockerd将根据`options`来初始化服务,服务端的命令和客户端一样也是采用`cobra`构建
1. 装载cobra命令模版`cmd/dockerd/newDaemonCommand`
2. 执行`cmd/dockerd/docker_unix.go/runDaemon`
3. 初始化各项参数,TLS、日志等级......
4. linux下检测`euid`是不是`root`
5. 设置umask
6. 创建docker根目录,默认`/var/lib/docker`,以及docker exec目录,默认`/var/run/docker`
7. 检测pid文件
8. 配置`API server`和监听端口
9. 初始化`ContainerD`以及退出清理函数
10. `NewDaemon`创建daemon对象
    ```go
        d, err := daemon.NewDaemon(ctx, cli.Config, pluginStore)
        if err != nil {
            return errors.Wrap(err, "failed to start daemon")
        }
    ```
11. 将`api server`的应答接口与daemon对象绑定,`initRouter(routerOptions)`

12. 通过`goroutine`启动`api servier`监听 
    ```go
        // The serve API routine never exits unless an error occurs
        // We need to start it as a goroutine and wait on it so
        // daemon doesn't exit
        serveAPIWait := make(chan error)
        go cli.api.Wait(serveAPIWait)
    ```
13. daemon结束会做`shutdownDaemon`停止进行中的容器

### NewDaemon
`NewDaemon`会进一步初始化`docker daemon`,并返回`Daemon`对象,Daemon对象将响应`api servier`执行具体的操作

1. 设置默认`mtu` 1500 
2. 初始化`registry service`
3. 对配置参数进行校验,对一些产生冲突的参数一起使用时报错并退出
4. 检测内核版本和系统
5. 创建`/var/lib/docker/containers`
6. 创建`/var/lib/docker/runtimes`, 初始化runtimes
7. 设置`grpc`,创建`containerdCli`
8. 创建插件管理器 
9. 创建`graphDrivers`
   ```go
    // List of drivers that should be used in an order
	priority = "btrfs,zfs,overlay2,fuse-overlayfs,aufs,overlay,devicemapper,vfs"

   ``` 
11. 创建volumeService
12. 创建imageService
13. 创建容器客户端libcontainerd

## moby源码编译
- v20.10.12
- aarch linux20.04

1. 工程目录下添加`sources.list`文件
    ```
    deb http://mirrors.163.com/debian/ buster main non-free contrib
    deb http://mirrors.163.com/debian/ buster-updates main non-free contrib
    deb http://mirrors.163.com/debian/ buster-backports main non-free contrib
    deb-src http://mirrors.163.com/debian/ buster main non-free contrib
    deb-src http://mirrors.163.com/debian/ buster-updates main non-free contrib
    deb-src http://mirrors.163.com/debian/ buster-backports main non-free contrib
    deb http://mirrors.163.com/debian-security/ buster/updates main non-free contrib
    deb-src http://mirrors.163.com/debian-security/ buster/updates main non-free contrib
    ```

2. 替换apt源，修改`Dockerfile`
    ```dockerfile
    #...
    ARG APT_MIRROR
    RUN sed -ri "s/(httpredir|deb).debian.org/${APT_MIRROR:-deb.debian.org}/g" /etc/apt/sources.list \
    && sed -ri "s/(security).debian.org/${APT_MIRROR:-security.debian.org}/g" /etc/apt/sources.list
    # 添加源
    ADD sources.list /etc/apt/sources.list
    ENV GO111MODULE=off
    #...
    ```
3. 修改`hack/dockerfile/install/containerd.installer`中git clone命令

    ```
    ...
    install_containerd() (
        echo "Install containerd version $CONTAINERD_VERSION"
        git clone https://ghproxy.com/https://github.com/containerd/containerd.git "$GOPATH/src/github.com/containerd/containerd"
    ...
    ```
4. `hack/dockerfile/install/proxy.installer`,`hack/dockerfile/install/runc.installer`,`hack/dockerfile/install/tini.installer`也同样修改`git clone`命令,在rul前面添加`https://ghproxy.com/`代理


5. `hack/dockerfile/install/rootlesskit.installer`添加`go env`代理
    ```
    _install_rootlesskit() (
        //...
	    export GOPROXY="https://goproxy.cn"
        for f in rootlesskit rootlesskit-docker-proxy; do
        //...
    )
    ```
6. `sudo make binary`,可执行文件在`bundles/binary-daemon`