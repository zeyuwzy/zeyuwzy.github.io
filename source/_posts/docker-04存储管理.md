---
title: Docker - 04存储管理
date: 2022-10-11 09:00:00
tags:
    - Docker
---

## 基本概念
`registry`是`repository`的集合，`repository`是镜像的集合

`Docker`内部的`image`(镜像)概念是用来存储一组镜像相关的元数据信息，主要包括镜像的架构、镜像默认配置信息、构建镜像的容器配置信息、包含所有镜像层信息的`rootfs`,`Docker`用`rootfs`中的`diff_id`计算出内容寻址的索引（chainID）来获取`layer`相关信息，进而获取每一个镜像层的文件内容

`layer`（镜像层）是一个`Docker`用来管理镜像层的中间概念，镜像是由镜像层组成的，而单个镜像层可能被多个镜像共享，所以`Docker`将`layer`与`image`的概念分离。`Docker`镜像管理中的`layer`主要存放了镜像层的`diff_id`、`size`、`cache-id`和`parent`等内容，实际的文件内容则是由存储驱动来管理，并可以通过`cache-id`在本地索引到

## repository元数据
`/var/lib/docker/image/overlay2/repositories.json`下存放所有镜像相关元数据(多个repository下的不同版本镜像),包含`repo`名称(docker-cli-dev),镜像名称和tag(docker-cli-dev:latest),镜像id(sha256...),有的还包含数字签名

Docker默认采用SHA256算法根据镜像元数据配置文件计算出镜像ID

```json
{
"Repositories":
{	
	"docker-cli-dev":
	{"docker-cli-dev:latest":"sha256:809278dd58853bdeba654f7b8605a792830d1e1ab4ba2c41a24b43c945998901"},

	"docker-cli-native":
	{"docker-cli-native:latest":"sha256:4bb4cbf4906ee605fb05d1b7d2e96e72dc2ee4703845431d910eb3c515b994f8"},

	"docker-dev":
	{"docker-dev:1-12-0":"sha256:0c12c9a82fe05a081fff33012812236c6ee84037e8375ba3d138df042efbbca9"},

	"redis":
	{"redis:latest":"sha256:e800a8da9469811a2d16e2f9aaa65374d33b04844209e85eda17986b21949a5c","redis@sha256:9bc34afe08ca30ef179404318cdebe6430ceda35a4ebe4b67d10789b17bdf7c4":"sha256:e800a8da9469811a2d16e2f9aaa65374d33b04844209e85eda17986b21949a5c"},

	"ubuntu":
	{"ubuntu:16.04":"sha256:fe3b34cb9255c8dd390f85c8fe37e6fbe6ac555f86c7e8944cd68da2de2f7633","ubuntu:latest":"sha256:bad148f8963fb9be6c8c260ce8a65aadd1cdfdd95e5bd16867d0f987cd7ebff3","ubuntu@sha256:34fea4f31bf187bc915536831fd0afc9d214755bf700b5cdb1336c82516d154e":"sha256:bad148f8963fb9be6c8c260ce8a65aadd1cdfdd95e5bd16867d0f987cd7ebff3","ubuntu@sha256:91bd29a464fdabfcf44e29e1f2a5f213c6dfa750b6290e40dd6998ac79da3c41":"sha256:fe3b34cb9255c8dd390f85c8fe37e6fbe6ac555f86c7e8944cd68da2de2f7633"}
	
}
}
```

## image元数据
`imageStore`则管理镜像ID与镜像元数据之间的映射关系以及元数据的持久化操作，持久化文件位于`/var/lib/docker/image/[graph_driver]/imagedb/content/sha256/[image_id]`中

内容包括镜像架构,构建镜像相关配置信息、创建该镜像的容器ID和配置、创建时间、创建该镜像的`Docker`版本、构建镜像的历史信息、操作系统以及`rootfs`组成

`Docker`会根据历史信息和`rootfs`中的`diff_ids`计算出构成该镜像的镜像层的存储索引`chainID`

## layer元数据
镜像层只包含一个具体的镜像层文件包,当一个镜像层被下载后,`Docker`会基于镜像层文件包和`image`元数据构建本地的`layer`元数据,包括diff、parent、size等

`Docker`中定义了`Layer`和`RWLayper`两种接口，分别用来定义只读层和可读写层的一些操作，又定义了`roLayer`和`mountedLayer`，分别实现了上述两种接口。其中，`roLayer`用于描述不可改变的镜像层，`mountedLayer`用于描述可读写的容器层

### roLayer
位于`/var/lib/docker/image/[graph_driver]/layerdb/sha256/[chainID]/`

- chainID 文件夹名称,用于索引该镜像层
  - 如果该镜像层是最底层(没有父镜像层),该层的`diffID`便是`chainID`
  - 该镜像层的`chainID`计算公式为`chainID(n)=SHA256(chain(n-1) diffID(n))`，也就是根据父镜像层的`chainID`加上一个空格和当前层的`diffID`，再计算`SHA256`校验码
- diff 该镜像层的校验码`diffID`
    - `diffID`采用`SHA256`算法，基于镜像层文件包的内容计算得到
- parent 父镜像层`chainID`
- cache-id `graphdriver`存储当前镜像层文件的`cacheID`
    - `cacheID`是在当前`Docker`宿主机上随机生成的一个`uuid`，在当前宿主机上与该镜像层一一对应，用于标示并索引`graphdriver`中的镜像层文件
- size 该镜像层的大小

### mountedLayer
位于`/var/lib/docker/image/[graph_driver]/layerdb/mounts/[container_id]/`
- container_id 对应`docker ps -a`的容器ID
- init-id 容器init层在graphdriver中的ID
- mount 读写层在graphdriver中的ID
- parent 父层镜像的chainID

```bash
root@zeyu-desktop:/var/lib/docker/image/overlay2/layerdb/mounts/f4ce0841c0b174e4e5e4639e1ef5841a14ebe64c37f3e6cf3711501567345842# ls
init-id  mount-id  parent
```


```bash
root@zeyu-desktop:/var/lib/docker/image/overlay2/layerdb/sha256/efc088ed50a3b76edfce1505de41e3317d521684dc0620f1cd87af5f6f28a9ab# pwd
/var/lib/docker/image/overlay2/layerdb/sha256/efc088ed50a3b76edfce1505de41e3317d521684dc0620f1cd87af5f6f28a9ab
root@zeyu-desktop:/var/lib/docker/image/overlay2/layerdb/sha256/efc088ed50a3b76edfce1505de41e3317d521684dc0620f1cd87af5f6f28a9ab# ls
cache-id  diff  parent  size  tar-split.json.gz
```