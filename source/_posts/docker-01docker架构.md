---
title: Docker - 01Docker架构
date: 2022-08-22 14:00:00
tags:
    - Docker
---

## 定义
根据官方的定义，`Docker`是以`Docker`容器为资源分割和调度的基本单位，封装整个软件运行时环境，为开发者和系统管理员设计的，用于构建、发布和运行分布式应用的平台。它是一个跨平台、可移植并且简单易用的容器解决方案

## docker命令
![](../photos/src/docker/01-docker命令.png)

![](../photos/src/docker/01-docker命令结构.jpeg)

`Docker`的设计理念是希望用户能够保证一个容器只运行一个进程，即只提供一种服务。然而，对于用户而言，单一容器是无法满足需求的。通常用户需要利用多个容器，分别提供不同的服务，并在不同容器间互连通信，最后形成一个`Docker`集群。`Docker`基于轻量级虚拟化技术，其易用、跨平台、可移植的特性使其在集群系统的搭建方面有着得天独厚的优势,因此`Docker`可以实现分布式应用集群的快速、准确、自动化部署

## docker架构
### docker client
任何遵循了`Docker API`的客户端都可以被称为`docker client`

```go
package main

import (
	"context"
	"fmt"

	"github.com/docker/docker/api/types"
	"github.com/docker/docker/client"
)

func getContainerList() error {
	cli, err := client.NewClientWithOpts(client.FromEnv)
	if err != nil {
		panic(err)
	}

	opts := types.ContainerListOptions{}
	opts.All = true

	containers, err := cli.ContainerList(context.Background(), opts)
	if err != nil {
		panic(err)
	}

	for _, container := range containers {
		fmt.Printf("%s %s\n", container.ID[:10], container.Image)
	}
	return err
}

func getImageList() error {
	cli, err := client.NewClientWithOpts(client.FromEnv)
	if err != nil {
		panic(err)
	}

	images, err := cli.ImageList(context.Background(), types.ImageListOptions{})
	if err != nil {
		panic(err)
	}

	for _, image := range images {
		fmt.Printf("%s ", image.ID)
		for _, val := range image.RepoTags {
			fmt.Printf("%s \n", val)
		}
	}

	return err
}

func main() {
	getContainerList()
	getImageList()
}
```

### docker daemon
是一个`API Server`,负责接收由`Docker client`发送的请求,统一分发调度

#### ImageService
实现了`Docker`镜像的管理

```go
// ImageService provides a backend for image management
type ImageService struct {
	containers                containerStore
	distributionMetadataStore metadata.Store
	downloadManager           *xfer.LayerDownloadManager
	eventsService             *daemonevents.Events
	imageStore                image.Store          
	layerStores               map[string]layer.Store // By operating system
	pruneRunning              int32
	referenceStore            dockerreference.Store
	registryService           registry.Service
	trustKey                  libtrust.PrivateKey
	uploadManager             *xfer.LayerUploadManager
	leases                    leases.Manager
	content                   content.Store
	contentNamespace          string
}
```
#### DistributionServices

负责镜像的存储

```go
// DistributionServices provides daemon image storage services
type DistributionServices struct {
	DownloadManager   distribution.RootFSDownloadManager
	V2MetadataService metadata.V2MetadataService
	LayerStore        layer.Store // TODO: lcow
	ImageStore        image.Store
	ReferenceStore    dockerreference.Store
}
```

#### dirver
1. `volumedriver`是`volume`数据卷存储操作的最终执行者，负责`volume`的增删改查,`docker`默认策略是将卷都存在本地的`Docker`根目录下的`volumes`
2. `graphdriver`是所有与容器镜像相关操作的最终执行者。`graphdriver`会在`Docker`工作目录下维护一组与镜像层对应的目录，并记下镜像层之间的关系以及与具体的`graphdriver`实现相关的元数据,这样，用户对镜像的操作最终会被映射成对这些目录文件以及元数据的增删改查，从而屏蔽掉不同文件存储实现对于上层调用者的影响,在`Linux`环境下，目前`Docker`已经支持的`graphdriver`包括`btrfs,zfs,overlay2,fuse-overlayfs,aufs,overlay,devicemapper,vfs`