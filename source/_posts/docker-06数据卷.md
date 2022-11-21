---
title: Docker - 06 数据卷
date: 2022-11-21 09:00:00
tags:
    - Docker
---

`volume`是存在于一个或多个容器中的特定文件或文件夹，这个目录以独立于联合文件系统的形式在宿主机中存在，并为数据的共享与持久化提供以下便利

```go
type Daemon struct {
    ...
	volumes           *volumesservice.VolumesService
    ...
}
```

- `volume`在容器创建时就会初始化，在容器运行时就可以使用其中的文件
- `volume`能在不同的容器之间共享和重用
- 对`volume`中数据的操作会马上生效
- 对`volume`中数据的操作不会影响到镜像本身
- `volume`的生存周期独立于容器的生存周期，即使删除容器，`volume`仍然会存在，没有任何容器使用的`volume`也不会被`Docker`删除

## 创建volume
```bash
docker volume create test_volume
```
`Docker`在创建`volume`的时候会在宿主机`/var/lib/docker/volume/`中创建一个以`volume ID`为名的目录，并将`volume`中的内容存储在名为`_data`的目录下

## 挂载volume
`docker`还支持直接将宿主机的一个目录挂载到容器中
```bash
docker run -it -v /tmp/abc:/data ubuntu /bin/bash
```

只读`volume`
```sh
docker run -it -v test_volume:/data:ro ubuntu /bin/bash
```

私有`volume`
```sh
docker run -it -v test_volume:/data:Z ubuntu /bin/bash
```

## 可以通过 volumes-from 参数实现容器间volume共享
## volume备份
通过临时的容器配合`--volumes-from`在容器内将需要备份的`volume`打包并导出到`backup`
```sh
docker run --rm --volumes-from vol_simple -v $(pwd):/backup ubuntu tar cvf /backup/data.tar /data​​
```

还原
```sh
docker run -it --name vol_bck -v /data ubuntu /bin/bash
docker run --rm --volumes-from vol_bck -v $(pwd):/backup ubuntu tar xvf /backup/data.tar -C /​
```

## 实现
数据卷都在`runc`基于`bind mount`实现,将需要绑定的原地址、目的地址写到`runtime spec`文件,`runc`在创建`rootfs`时会将对应地址绑定到新的`rootfs`然后支持`pivot_root`