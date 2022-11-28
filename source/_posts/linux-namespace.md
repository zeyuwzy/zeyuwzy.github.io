---
title: Namespace
date: 2022-11-28 08:49:11
tags:
    - Linux
    - Linux Namespace
---

## 概念

`Linux namespaces`将全局系统资源以抽象的方式封装隔离，这使得命名空间中的进程似乎拥有独立的全局系统资源实例。改变一个`namespace`中的系统资源只会影响当前`namespace`里的进程，对其他`namespace`中的进程没有影响

## 命名空间类型

| 命名空间 | 标志 | man文档 | 隔离内容 |
|--|--|--|--|
| Cgroup| CLONE_NEWCGROUP | cgroup_namespaces(7) | Cgroup root directory (`cgroup`根目录）|
| IPC | CLONE_NEWIPC | ipc_namespaces(7) | System V IPC(信号量、消息队列、共享内存), POSIX message queues |
| Network | CLONE_NEWNET | network_namespaces(7) | Network devices,stacks, ports, etc. (网络设备、网络栈、端口) |
| Mount |    CLONE_NEWNS  |   mount_namespaces(7) |  Mount points (挂载点) |
| PID  |     CLONE_NEWPID |   pid_namespaces(7)   |  Process IDs (进程号) |
| User |     CLONE_NEWUSER |  user_namespaces(7)  |  User and group IDs (用户和组） |
| UTS |      CLONE_NEWUTS  |  uts_namespaces(7)  |   Hostname and NIS domain name (主机名、`NIS`域名)|

## 命名空间相关API

- **clone** 可以创建一个子进程，可以同时通过`flags`参数创建对应命名空间
- **setns** 可以让调用的进程通过打开`/proc/[pid]/ns`文件描述符加入一个已经存在的命名空间
- **unshare** 不启动新的进程,直接让调用进程加入新的命名空间
- **ioctl/ioctl_ns** 获取命名空间信息

一般情况下使用`clone`,或者`unshare`创建新命名空间都需要`CAP_SYS_ADMIN`能力,自从`linux 3.8`支持了非特权用户创建用户命名空间

命名空间的创建者有能力去修改全局资源达到控制后续创建的进程以及`join`到这个命名空间的进程在这个命名空间资源访问

## /proc/[pid]/ns/
这个目录下包含进程对应的每一个命名空间
```sh
zeyu@zeyu-desktop:~$ ls -l /proc/$$/ns
total 0
lrwxrwxrwx 1 zeyu zeyu 0 Nov 28 15:49 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 zeyu zeyu 0 Nov 28 15:49 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 zeyu zeyu 0 Nov 28 15:49 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx 1 zeyu zeyu 0 Nov 28 15:49 net -> 'net:[4026531897]'
lrwxrwxrwx 1 zeyu zeyu 0 Nov 28 15:49 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 zeyu zeyu 0 Nov 28 15:49 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 zeyu zeyu 0 Nov 28 15:49 user -> 'user:[4026531837]'
lrwxrwxrwx 1 zeyu zeyu 0 Nov 28 15:49 uts -> 'uts:[4026531838]'
```

将对应的文件保持打开或者将文件通过`bind`绑定到其他地方可以让对应命名空间一直存在,即使命名空间内的进程都已经退出

如果两个进程在同一个命名空间,那么他们对应节点的设备`id`,和`inode`是相同的,可以通过`stat.st_dev`和`stat.st_ino`查看

软连接指向内容是命名空间的类型和对应的`inode`
```sh
zeyu@zeyu-desktop:/proc/self/ns$ readlink /proc/self/ns/cgroup 
cgroup:[4026531835]
```

## /proc/sys/user (linux 4.9)
包含每一个命名空间可以被创建的最大限制
```sh
zeyu@zeyu-desktop:/proc/self/ns$ ll /proc/sys/user/
total 0
dr-xr-xr-x 1 root root 0 Nov 28 16:12 ./
dr-xr-xr-x 1 root root 0 Nov 28 11:03 ../
-rw-r--r-- 1 root root 0 Nov 28 16:12 max_cgroup_namespaces
-rw-r--r-- 1 root root 0 Nov 28 16:12 max_inotify_instances
-rw-r--r-- 1 root root 0 Nov 28 16:12 max_inotify_watches
-rw-r--r-- 1 root root 0 Nov 28 16:12 max_ipc_namespaces
-rw-r--r-- 1 root root 0 Nov 28 16:12 max_mnt_namespaces
-rw-r--r-- 1 root root 0 Nov 28 16:12 max_net_namespaces
-rw-r--r-- 1 root root 0 Nov 28 16:12 max_pid_namespaces
-rw-r--r-- 1 root root 0 Nov 28 16:12 max_user_namespaces
-rw-r--r-- 1 root root 0 Nov 28 16:12 max_uts_namespaces
```

- 只有特权用户能修改
- 限制是围绕进程所在**用户命名空间**展开的
- 限制是基于用户的,同一用户命名空间下的每一个用户都可以创建其他命名空间直到上限
- 限制将针对所有用户, 包括`uid=0` 
- `clone`,`unshare`因达到上限失败返回`ENOSPC`
- 当一个`uid`创建子**用户命名空间**, 子用户命名空间下创建命名空间时, 也会根据`uid`累计到所有祖先命名空间。确保创建新的用户命名空间不会做为逃避限制的手段

## 命名空间生命周期
如果没有任何其他因素，当命名空间中的最后一个进程终止或离开命名空间时，命名空间将被自动删除。然而即使命名空间没有成员进程，也有许多其他因素可能会导致命名空间存在

* 对应的`/proc/[pid]/ns/*`文件存在打开的文件描述符或`bind`
* 命名空间是分层的（即`pid`或`user`名空间）,并且具有子命名空间
* `user namespace`,拥有一个或多个其他非用户命名空间
* `PID namespace`,有一个进程通过`/proc/[PID]/ns/PID_for_children`符号链接引用该名称空间
* `ipc namespace`,`mqueue`文件系统的`mount`引用了这个命名空间
* `pid namespace`, `proc`文件系统的`mount`引用了这个命名空间



