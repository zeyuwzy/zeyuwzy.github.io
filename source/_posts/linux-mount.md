---
title: Linux mount
date: 2022-11-26 10:00:00
tags:
    - Linux
---
## 基本概念

```sh
mount -t type device dir
```

`mount`命令用于将某些设备上的文件系统附加到指定目录,`dir`即是对应文件系统的根目录,`mount`后原目录下的内容都将不可见

多数情况下不用指定`-t`参数显式地说明文件系统的类型

通过查看`/proc/filesystems`文件来查看当前系统具体都支持哪些文件系统,`nodev`表示不需要设备

```sh
zeyu@zeyu-desktop:/proc$ cat filesystems 
nodev	sysfs
nodev	tmpfs
nodev	bdev
nodev	proc
nodev	cgroup
nodev	cgroup2
nodev	cpuset
nodev	devtmpfs
nodev	configfs
nodev	debugfs
nodev	tracefs
nodev	securityfs
nodev	sockfs
nodev	bpf
nodev	pipefs
nodev	ramfs
nodev	hugetlbfs
nodev	devpts
	ext3
	ext2
	ext4
	squashfs
	vfat
nodev	ecryptfs
	fuseblk
nodev	fuse
nodev	fusectl
nodev	efivarfs
nodev	mqueue
nodev	pstore
nodev	autofs
nodev	binfmt_misc
nodev	overlay
nodev	prl_fs
nodev	aufs
```

同一个文件系统可以被挂载多次,甚至可以在同一个挂载点被挂载多次,这些完全是由内核以及不同文件系统特性决定的

## 列表显示挂载信息
```bash
mount [-l] [-t type]
```
更推荐使用`findmnt`

## 设备标识 
磁盘分区的设备名称不稳定；硬件重新配置、添加或删除设备可能会导致名称更改。强烈建议使用`UUID`或`LABEL`等标识符

命令`lsblk --fs`提供了可用块设备上的文件系统、`LABEL`和`UUID`的概述

命令`blkid -p ＜device＞`提供了有关指定设备上文件系统的详细信息

无法保证`UUID`和标签真的是唯一的，尤其是当移动、共享或复制设备时。使用`lsblk -o +UUID,PARTUUID`以验证`UUID`在系统中是否真正唯一

## /etc/fstab, /etc/mtab and /proc/mounts

`/etc/mtab`和 `/proc/mounts`一样，都是指向`/proc/self/mounts`的链接文件,用于维护当前挂载的文件系统的列表同时支持命名空间，容器和其他高级Linux功能一起使用
```c
struct mntent
  {
    char *mnt_fsname;           /* 文件系统对应的设备路径或者服务器地址  */
    char *mnt_dir;              /* 文件系统挂载到的系统路径 */
    char *mnt_type;             /* 文件系统类型: ufs, nfs, 等  */
    char *mnt_opts;             /* 文件系统挂载参数，以逗号分隔  */
    int mnt_freq;               /* 文件系统备份频率（以天为单位）  */
    int mnt_passno;             /* 开机fsck的顺序，如果为0，不会进行check */
  };
```

挂载`fstab`或`mtab`中提到的文件系统时，只需在命令行上仅指定设备或仅指定挂载点即可

`mount /dir`会让`mount`去`/etc/fstab`下寻找定义的挂载点,`mount dev`也是同理,可以通过`--target`或者`--source`避免歧义

`mount -a` 会将`/etc/fstab`中定义的所有挂载点都挂上,一般是在系统启动时的脚本中调用

如果要覆盖`/etc/fstab`中的安装选项，则必须使用`-o`选项
```bash
    mount device|dir -o options
```
如果同时指定了设备（或`LABEL`，`UUID`，`PARTUUID`或`PARTLABEL`）和目录，则挂载程序不会读取`/etc/fstab`文件

可以通过命令行选项`–options-source-force`更改此默认行为，以始终从`fstab`读取配置。对于非`root`用户，`mount`始终读取`fstab`配置


## 非特权用户
通常只有超级用户才能挂载文件系统。但是，当`fstab`在一行上包含`user`选项时，任何人都可以挂载相应的文件系统

挂载文件系统对应的用户才能卸载它。如果有任何用户应该能够卸载它，则在`fstab`行中使用`users`而不是`user`

`owner`选项限制用户必须是挂载文件的所有者,`group`选项也是同理


## 挂载
### 只读挂载
```sh
mount -o ro /dev/sdb1 /mnt
```
### 重新挂载
```sh
mount /mnt -o rw,remount
```
### 挂载虚拟文件系统
`proc`、`tmpfs`、`sysfs`、`devpts`等都是`Linux`内核映射到用户空间的虚拟文件系统，它们不和具体的物理设备关联，但它们具有普通文件系统的特征，应用层程序可以像访问普通文件系统一样来访问他们

由于是内核虚拟的一个文件系统，并没有对应的设备，所以这里的`-t`参数不能省略。由于没有对应的源设备，这里的设备可以是任意字符串，取个有意义的名字就可以了，因为用`mount`命令查看挂载点信息时第一列显示的就是这个字符串

```sh
mount -t tmpfs -o size=512m tmpfs /mnt
```

### 挂载loop设备
`loop device`是虚拟的块设备，主要目的是让用户可以像访普通块设备那样访问一个文件。`loop device`设备的路径一般是`/dev/loop0、dev/loop1`等

需要用到`loop device`的最常见的场景是挂载一个`ISO`文件。比如将`/tmp/test.iso`这个光盘镜像文件使用`loop`模式挂载到`/mnt`下，这样就不需要把`ISO`文件刻录到光盘上了，当然也不需要光驱了
```sh
mkisofs -o test.iso projects/
mount test.iso /mnt
```

创建虚拟硬盘
```sh
dd if=/dev/zero bs=1M count=512 of=./vdisk.img
mkfs -t ext4 vdisk.img
mount vdisk.img /mnt
```


## bind
可以将任何一个挂载点、普通目录或者文件挂载到其它的地方

绑定挂载并不会处理子挂载点,需要使用`--rbind`

只读`bind`
```sh
mount -o bind,ro foo foo
```

但是内核并不支持直接只读`bind`,所以上述命令并不是一个原子操作,而是通过`remount`实现的
```sh
    mount --bind olddir newdir
    mount -o remount,bind,ro olddir newdir
```

同样的`nosuid`, `nodev`, `noexec`, `noatime`, `nodiratime`, `relatime`等`vfs`标志也可以通过这个方式添加,但是像这样`-o rbind,ro`递归添加参数是不可以的

## move
把旧的挂载点移动到新的目录去
```sh
mount --move olddir newdir
```

果挂载点的父挂载点为`shared`类型，就不能移动该挂载点,可以通过`findmnt -o TARGET,PROPAGATION`查看所有挂载点传播标志

可以通过`mount --make-private /`修改父挂载点传播类型为`private`,然后再用`findmnt -o TARGET,PROPAGATION /`确认

**请求--make-操作时，mount不会读取fstab,必须在命令行上指定所有必需的信息**

## Shared subtree
`shared subtree`就是一种控制子挂载点能否在其他地方被看到的技术，它只会在`bind mount`和`mount namespace`中用到。

移动挂载点时提到了挂载点的类型，其实叫`propagation type`(传播类型)。

### propagation type
每个挂载点都有一个`propagation type`标志，由它来决定当在一个挂载点的下面创建和移除挂载点的时候，是否会传播到属于相同`peer group`的其他挂载点下面，也即同一个`peer group`里的其他的挂载点下面是不是也会创建和移除相应的挂载点。当前一共有4种不同类型的 `propagation type`

- **shared**：从名字就可以看出，挂载信息会在同一个`peer group`的不同挂载点之间共享传播。当一个挂载点下面添加或者删除挂载点的时候，同一个`peer group`里的其他挂载点下面也会挂载和卸载同样的挂载点
- **private**：跟上面的刚好相反，挂载信息根本就不共享，也即`private`的挂载点不会属于任何`peer group`
- **slave**：信息的传播是单向的，在同一个`peer group`里面，`master`的挂载点下面发生变化的时候，`slave`的挂载点下面也跟着变化。但反之则不然，`slave`下发生变化的时候不会通知`master`，`master`不会发生变化
- **unbindable**：这个和`private`相同，只是这种类型的挂载点不能作为`bind mount`的源，主要用来防止递归嵌套情况的出现

`propagation type`是挂载点的属性，对每个挂载点来说都是独立的

挂载点是有父子关系的，比如挂载点`/`和`/mnt`，`/mnt`是`/`的子挂载点，`/`是`/mnt`的父挂载点

默认情况下，如果父挂载点是`shared`，那么子挂载点也是`shared`的

### peer group
`peer group`就是一个或多个挂载点的集合，他们之间可以共享挂载信息。在下面两种情况下会使两个挂载点属于同一个`peer group`(前提条件是**挂载点**的`propagation type`是`shared`)

- 利用`mount --bind`命令，将会使源和目标挂载点属于同一个`peer group`，当然前提条件是源必须要是一个挂载点
- 当创建新的`mount namespace`时，新`namespace`会拷贝一份老`namespace`的挂载点信息，于是新的和老的`namespace`里面的相同挂载点就会属于同一个`peer group`

`propagation type`和`peer group`表现在`/proc/self/mountinfo`的`(7)`中,注意这个字段可能为空(当挂载方式为`--make-private`)

(`master:1`表示当前挂载点是`peer group 1`的`slave`)

```
See http://man7.org/linux/man-pages/man5/proc.5.html

36 35 98:0 /mnt1 /mnt2 rw,noatime master:1 - ext3 /dev/root rw,errors=continue
(1)(2)(3)   (4)   (5)      (6)      (7)   (8) (9)   (10)         (11)

(1) mount ID:  unique identifier of the mount (may be reused after umount)
(2) parent ID:  ID of parent (or of self for the top of the mount tree)
(3) major:minor:  value of st_dev for files on filesystem
(4) root:  root of the mount within the filesystem
(5) mount point:  mount point relative to the process's root
(6) mount options:  per mount options
(7) optional fields:  zero or more fields of the form "tag[:value]"
(8) separator:  marks the end of the optional fields
(9) filesystem type:  name of filesystem of the form "type[.subtype]"
(10) mount source:  filesystem specific information or "none"
(11) super options:  per super block options

```

