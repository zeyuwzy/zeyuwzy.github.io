---
title: Docker - 04存储管理
date: 2022-10-11 09:00:00
tags:
    - Docker
---

## 基本概念
`registry`用以保存`Docker`镜像，其中还包括镜像层次结构和关于镜像的元数据(Docker Hub)

`repository`即由具有某个功能的`Docker`镜像的所有迭代版本构成的镜像组

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

## graphdriver
```go
type Driver interface {
	ProtoDriver //定义了driver基本的功能
	DiffDriver //对数据层之间的差异（diff）进行管理
}
```

```go
type ProtoDriver interface {
	// String returns a string representation of this driver.
    // 返回一个代表这个驱动的字符串，通常是这个驱动的名字
	String() string
	// CreateReadWrite creates a new, empty filesystem layer that is ready
	// to be used as the storage for a container. Additional options can
	// be passed in opts. parent may be "" and opts may be nil.
	CreateReadWrite(id, parent string, opts *CreateOpts) error
	// Create creates a new, empty, filesystem layer with the
	// specified id and parent and options passed in opts. Parent
	// may be "" and opts may be nil.
    // 创建一个新的镜像层，需要调用者传进一个唯一的ID和所需的父镜像的ID
	Create(id, parent string, opts *CreateOpts) error
	// Remove attempts to remove the filesystem layer with this id.
    // 根据指定的ID删除一个层
	Remove(id string) error
	// Get returns the mountpoint for the layered filesystem referred
	// to by this id. You can optionally specify a mountLabel or "".
	// Returns the absolute path to the mounted layered filesystem.
    // 返回指定ID的层的挂载点的绝对路径
	Get(id, mountLabel string) (fs containerfs.ContainerFS, err error)
	// Put releases the system resources for the specified id,
	// e.g, unmounting layered filesystem.
    // 释放一个层使用的资源，比如卸载一个已经挂载的层
	Put(id string) error
	// Exists returns whether a filesystem layer with the specified
	// ID exists on this driver.
    // 查询指定的ID对应的层是否存在
	Exists(id string) bool
	// Status returns a set of key-value pairs which give low
	// level diagnostic status about this driver.
    // 返回这个驱动的状态，这个状态用一些键值对表示
	Status() [][2]string
	// Returns a set of key-value pairs which give low level information
	// about the image/container driver is managing.
    // 返回这个驱动正在管理的信息,用键值对表示
	GetMetadata(id string) (map[string]string, error)
	// Cleanup performs necessary tasks to release resources
	// held by the driver, e.g., unmounting all layered filesystems
	// known to this driver.
    // 释放由这个驱动管理的所有资源，比如卸载所有的层
	Cleanup() error
}
```

```go
// DiffDriver is the interface to use to implement graph diffs
type DiffDriver interface {
	// Diff produces an archive of the changes between the specified
	// layer and its parent layer which may be "".
	// 将指定ID的层相对父镜像层改动的文件打包并返回
	Diff(id, parent string) (io.ReadCloser, error)
	// Changes produces a list of changes between the specified layer
	// and its parent layer. If parent is "", then all changes will be ADD changes.
	// 返回指定镜像层与父镜像层之间的差异列表
	Changes(id, parent string) ([]archive.Change, error)
	// ApplyDiff extracts the changeset from the given diff into the
	// layer with the specified id and parent, returning the size of the
	// new layer in bytes.
	// The archive.Reader must be an uncompressed stream.
	// 从差异文件包中提取差异列表，并应用到指定ID的层与父镜像层，返回新镜像层的大小
	ApplyDiff(id, parent string, diff io.Reader) (size int64, err error)
	// DiffSize calculates the changes between the specified id
	// and its parent and returns the size in bytes of the changes
	// relative to its base filesystem directory.
	// 计算指定ID层与其父镜像层的差异，并返回差异相对于基础文件系统的大小
	DiffSize(id, parent string) (size int64, err error)
}
```
`Docker`中的任何存储驱动都需要实现上述`Driver`接口
`linux`系统下驱动的优先级被定义在`graphdriver/drivier_linux.go`中
```go
priority = "btrfs,zfs,overlay2,fuse-overlayfs,aufs,overlay,devicemapper,vfs"
```

### 驱动注册
`graphdriver`会维护一个`map`用于存储注册的驱动名称和对应的初始化函数
```go
var (
	// All registered drivers
	drivers map[string]InitFunc
)
```

以`overlay2`为例,只需要实现一个`init`函数

```go
func init() {
	graphdriver.Register(driverName, Init)
}
```
当`overlay2`的包被包含的时候自动就会调用包中的`init`函数,所有注册的驱动都在`graphdriver/register`,其他模块使用`register`包就会把相关的驱动注册

这里`import`时在包路径前面加了一个`_`,只有这样才能调用到相关包的`init`函数
```go
package register // import "github.com/docker/docker/daemon/graphdriver/register"

import (
	// register the overlay2 graphdriver
	_ "github.com/docker/docker/daemon/graphdriver/overlay2"
)
```

### 创建Driver
检查环境变量`DOCKER_DRIVER`和配置是否提供驱动名称,如果有的话直接在`map`按驱动名称调用对应驱动的初始化函数;否则就会按照优先级去查找相关的驱动

`Docker`镜像管理部分与存储驱动在设计上完全分离了，镜像层或者容器层在存储驱动中拥有一个新的标示`ID`，在镜像层（roLayer）中称为`cacheID`，容器层（mountedLayer）中为`mountID`。在`Unix`环境下，`mountID`是随机生成的并保存在`mountedLayer`的元数据`mountID`中,持久化文件在`/var/lib/docker/image/overlay2/layerdb/mounts/container_id`

### overlay2 driver
`OverlayFS`是一种新型联合文件系统（union filesystem），它允许用户将一个文件系统与另一个文件系统重叠（overlay），在上层的文件系统中记录更改，而下层的文件系统保持不变

```sh
mkdir lower upper work merged
mount -t overlay overlay￼-o lowerdir=./lower, upperdir=./upper, workdir=./work ./merged
```
`/var/lib/docker/overlay2`中
- 容器层
	- `mountID`
		```bash
		root@zeyu-desktop:/var/lib/docker/overlay2/89610a9bd010adaa7f2281c5b3c9d96d337ff87660411539a58f163d3cec1704# ls
		diff  link  lower  work
		```	
		- diff 存放容器运行时变化的文件(overlay的upper)
		- link 描述该层对应的软连接 所有的链接文件都在`/var/lib/docker/overlay2/l/`,使用软连接可以防止`lower`层过多时命令过长挂载失败
		- lower 描述该层对应所有的`lower`层的软连接
		- work `overlay`工作目录
		- merged 只有在运行时创建的`overlay`挂载点
	- `mountID-init`
		```bash
		root@zeyu-desktop:/var/lib/docker/overlay2/89610a9bd010adaa7f2281c5b3c9d96d337ff87660411539a58f163d3cec1704-init# ls
		committed  diff  link  lower  work
		```
- 镜像层
	- `cacheID`
		```bash
		root@zeyu-desktop:/var/lib/docker/overlay2/47bdd7a31bb6ad0f91d51d1c3b340e382d4a1427243eab85e4ec6ee1d9fb6d44# ls
		committed  diff  link
		```

```bash

mount

overlay on /var/lib/docker/overlay2/89610a9bd010adaa7f2281c5b3c9d96d337ff87660411539a58f163d3cec1704/merged type overlay (rw,relatime,lowerdir=/var/lib/docker/overlay2/l/VADBLOWNHH7XWGLDL2LLUS4LUU:/var/lib/docker/overlay2/l/D3ZMYBHFKVFZSLDBXLD3T6I26O,upperdir=/var/lib/docker/overlay2/89610a9bd010adaa7f2281c5b3c9d96d337ff87660411539a58f163d3cec1704/diff,workdir=/var/lib/docker/overlay2/89610a9bd010adaa7f2281c5b3c9d96d337ff87660411539a58f163d3cec1704/work,xino=off)
```