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
- 装载cobra命令模版`cmd/dockerd/docker.go/newDaemonCommand`
- 执行`cmd/dockerd/docker_unix.go/runDaemon`
- 初始化各项参数,TLS、日志等级......
- linux下检测`euid`是不是`root`
- 设置umask
- 创建docker根目录,默认`/var/lib/docker`,以及`docker exec`目录,默认`/var/run/docker`
- 检测pid文件
- 配置`API server`和监听端口
- 初始化`ContainerD`以及退出清理函数
- `NewDaemon`创建daemon对象
    ```go
        d, err := daemon.NewDaemon(ctx, cli.Config, pluginStore)
        if err != nil {
            return errors.Wrap(err, "failed to start daemon")
        }
    ```
- 将`api server`的应答接口与daemon对象绑定,`initRouter(routerOptions)`

- 通过`goroutine`启动`api servier`监听 
    ```go
        // The serve API routine never exits unless an error occurs
        // We need to start it as a goroutine and wait on it so
        // daemon doesn't exit
        serveAPIWait := make(chan error)
        go cli.api.Wait(serveAPIWait)
    ```
- daemon结束会做`shutdownDaemon`停止进行中的容器

### NewDaemon
`NewDaemon`会进一步初始化`docker daemon`,并返回`Daemon`对象,Daemon对象将响应`api servier`执行具体的操作

- 设置默认`mtu` 1500 
- 初始化`registry service`
- 对配置参数进行校验,对一些产生冲突的参数一起使用时报错并退出
- 检测内核版本和系统
- 创建`/var/lib/docker/containers`
- 创建`/var/lib/docker/runtimes`, 初始化runtimes
- 设置`grpc`,创建`containerdCli`
- 创建插件管理器 
- 创建`graphDrivers`
   ```go
    // List of drivers that should be used in an order
	priority = "btrfs,zfs,overlay2,fuse-overlayfs,aufs,overlay,devicemapper,vfs"

   ``` 
- 创建volumeService
- 创建imageService
- 创建容器客户端libcontainerd

## 创建容器流程

`docker create`

- 当客户端发起创建容器，`daemon`接收请求后创建容器在`daemon/creat.go/ContainerCreate`
- 通过`imageService`获取创建容器对应的镜像,并提取镜像`ID`
    ```go
		img, err = daemon.imageService.GetImage(opts.params.Config.Image, opts.params.Platform)
    ```
- 通过`newContainer`函数创建`Container`结构 
    - 创建容器`ID`和名称`generateIDAndName`
- 通过`imageService`的`CreateLayer`函数创建容器在`graphdriver`上的`init`层(daemon/initlayer/setup_unix.go/Setup)和读写层
- 创建容器根目录`/var/lib/docker/containers/container-id`
- 在容器根目录创建`checkpoint`和容器相关配置文件信息
- 向`daemon`注册该容器
- 将容器状态设置为`stopped`
- 函数调用成功后会返回容器`ID`和创建时的一些警告

## 启动容器流程

`docker start`

- 当客户端发起启动容器流程时,`daemon`接收请求后执行容器核心流程在`daemon/start.go/ContainerStart`
- 根据容器的`ID`获取`Container`数据结构
- 检查要运行的容器是否处于暂停、已经执行、被移除的状态
- 对参数`ctr.HostConfig`进行验证
- 挂载容器读写层到`merge`
    ```go
    	if err := daemon.conditionalMountOnStart(container); err != nil {
    		return err
    	}
    ```
- 初始化网络相关配置`hosts`、`hostname`、`resolv.conf`
- 后面就是和`containerd`交互
- 基于容器的相关参数去初始化`oci`(Open Container Initiative)`runtime-spec`项目的`Spec`数据结构
- 初始化`ShimConfig`,`containerd`将通过`shim`去调用`runc`
- 执行`daemon.containerd.Create`创建容器
- 执行`daemon.containerd.Start`启动容器
- 设置容器的状态

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