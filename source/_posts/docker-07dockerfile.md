---
title: Docker - 07 dockerfile
date: 2022-11-22 09:00:00
tags:
    - Docker
---

## 说明
`Dockerfile`是一个用来构建镜像的文本文件

```dockerfile
ARG GO_VERSION=1.18.5

FROM    golang:${GO_VERSION}-alpine

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

RUN     apk add -U git bash coreutils gcc musl-dev

ENV     CGO_ENABLED=0 \
        DISABLE_WARN_OUTSIDE_CONTAINER=1
WORKDIR /go/src/github.com/docker/cli
CMD     ./scripts/build/binary

```

## 命令
### FROM
构建镜像基于哪个基础镜像

```dockerfile
FROM [--platform=<platform>] <image> [AS <name>]
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]

```

- 单个`dockfile`可以多次出现`FROM`,以使用之前的构建阶段作为另一个构建阶段的依赖项
- `AS name`表示为构建阶段命名，在后续`FROM`和`COPY --from=<name>`说明中可以使用这个名词，引用此阶段构建的映像
- tag或digest值是可选的。如果您省略其中任何一个，构建器默认使用一个latest标签。如果找不到该tag值，构建器将返回错误。
- --platform标志可用于在FROM引用多平台镜像的情况下指定平台。例如，linux/amd64、linux/arm64、或windows/amd64

### RUN
将在当前镜像之上的新层中执行命令，在`docker build`时运行

RUN有两种形式：
```dockerfile
RUN <command>（shell形式，命令在shell中运行，默认/bin/sh -c在Linux或cmd Windows上）
RUN ["executable", "param1", "param2"]（执行形式）
```
- 可以使用\（反斜杠）将单个`RUN`指令延续到下一行
- RUN在下一次构建期间,指令缓存不会自动失效。可以使用--no-cache标志使指令缓存无效
- `Dockerfile`的指令每执行一次都会在`docker`上新建一层。所以过多无意义的层,会造成镜像膨胀过大,可以使用`&&`符号连接命令,这样执行后,只会创建1层镜像

### CMD
运行程序，在docker run时运行

RUN有三种形式：
```dockerfile
CMD ["executable","param1","param2"] 使用exec执行,推荐方式
CMD command param1 param2 在 /bin/sh 中执行,提供给需要交互的应用
CMD ["param1","param2"] 提供给 ENTRYPOINT 的默认参数
```

- 指定启动容器时执行的命令,每个`Dockerfile`只能有一条`CMD`命令。如果指定了多条命令,只有最后一条会被执行
- 如果用户启动容器时候指定了运行的命令,则会覆盖掉`CMD`指定的命令

### ENTRYPOINT
ENTRYPOINT 和 CMD 一样，都是在指定容器启动程序及参数，不过它不会被 docker run 的命令行参数指定的指令所覆盖。如果要覆盖的话，需要通过docker run --entrypoint 来指定。

ENTRYPOINT有两种格式

```dockerfile
ENTRYPOINT ["executable", "param1", "param2"]
ENTRYPOINT command param1 param2
```
如果 Dockerfile 中如果存在多个 ENTRYPOINT 指令，仅最后一个生效。

当指定了 ENTRYPOINT 后， CMD 的内容作为参数传给 ENTRYPOINT 指令，实际执行时，将变为：
```dockerfile
<ENTRYPOINT> <CMD>
```
### ARG
定义变量,ARG设置的环境变量仅对Dockerfile内有效
```dockerfile
ARG <name>[=<default value>]
```
docker build 中可以用 --build-arg <参数名>=<值> 来覆盖

内置ARG
- HTTP_PROXY
- http_proxy
- HTTPS_PROXY
- https_proxy
- FTP_PROXY
- ftp_proxy
- NO_PROXY
- no_proxy

### ENV

```dockerfile
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
```
设置的环境变量将持续存在，可以使用docker inspect来查看

使用`docker run --env <key>=<value>`来更改环境变量的值

如果环境变量只在构建期间需要
```dockerfile
RUN DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -y ...
```
或者
```dockerfile
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y ...
```

### COPY
复制指令,从上下文目录(docker build 传入)中复制文件或者目录到容器里指定路径
```dockerfile
COPY [--chown=<user>:<group>] <源路径1>...  <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",...  "<目标路径>"]
```
`[--chown=<user>:<group>]`：可选参数,用户改变复制到容器内文件的拥有者和属组
目标路径不存在的话,会自动创建

### ADD
和copy差不多
- add 支持src为远程url
- add 支持将src为tar的压缩文件解压到dst



### WORKDIR
指定工作目录。用`WORKDIR`指定的工作目录，会在构建镜像的每一层中都存在

### LABEL
添加元数据(metadata),键值对
```dockerfile
LABEL org.authors="zeyu"
```

### EXPOSE
Docker容器在运行时声明网络端口。可以指定端口是监听TCP还是UDP，如果不指定协议，默认为TCP

在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口

```
EXPOSE 80/udp
```

### VOLUME
定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷
```dockerfile
VOLUME ["/var/log/"]
VOLUME /var/log
```

`Dockerfile`中除了`FROM`指令的每一行都是基于上一行生成的临时镜像运行一个容器，执行一条指令并执行类似`docker commit`的命令得到一个新的镜像，这条类似`docker commit`的命令不会对挂载的`volume`进行保存


```dockerfile
​​FROM ubuntu
RUN useradd foo
VOLUME /data
RUN touch /data/file
RUN chown -R foo:foo /data
```

需要将`VOLUME /data`放到最后一行执行,在挂载`volume`时，`/data`已经存在，`/data`中的文件以及它们的权限和所有者设置会被复制到`volume`中

### ONBUILD
将一个触发指令添加到镜像中，以便稍后在该镜像用作另一个构建的基础时执行。也就是另外一个dockerfile FROM了这个镜像的时候执行
```dockerfile
ONBUILD <其它指令>
```

