---
title: Linux Cgroups
date: 2022-12-02 10:00:00
tags:
    - Linux
---

## 概念

`cgroups`是`Linux`内核提供的一种机制，这种机制可以根据需求把一系列系统任务及其子任务整合（或分隔）到按资源划分等级的不同组内，从而为系统资源管理提供一个统一的框架。

本质上来说，`cgroups`是内核附加在程序上的一系列钩子（hook），通过程序运行时对资源的调度触发相应的钩子以达到资源追踪和限制的目的。

### 特性

* `cgroups`的`API`以一个伪文件系统(cgroupfs)的方式实现，用户态的程序可以通过文件操作实现`cgroups`的组织管理
* `cgroups`的组织管理操作单元可以细粒度到线程级别，另外用户可以创建和销毁`cgroup`，从而实现资源再分配和管理
* 所有资源管理的功能都以子系统的方式实现，接口统一
* 子任务创建之初与其父任务处于同一个`cgroups`的控制组

### 功能

* **资源限制**：`cgroups`可以对任务使用的资源总额进行限制
* **优先级分配**：通过分配的`CPU`时间片数量及磁盘`IO`带宽大小，实际上就相当于控制了任务运行的优先级
* **资源统计**：`cgroups`可以统计系统的资源使用量，如`CPU`使用时长、内存用量等，这个功能非常适用于计费
* **任务控制**：`cgroups`可以对任务执行挂起、恢复等操作

### 名词

* **Task**(任务)：在`linux`系统中，内核本身的调度和管理并不对进程和线程进行区分，只是根据`clone`时传入的参数的不同来从概念上区分进程和线程。这里使用`task`来表示系统的一个进程或线程
* **Cgroup**(控制组)：`cgroups`中的资源控制以`cgroup`为单位实现。`Cgroup`表示按某种资源控制标准划分而成的任务组，包含一个或多个子系统。一个任务可以加入某个`cgroup`，也可以从某个`cgroup`迁移到另一个`cgroup`
* **Subsystem**(子系统)：`cgroups`中的子系统就是一个资源调度控制器(又叫`controllers`)。比如`CPU`子系统可以控制`CPU`的时间分配，内存子系统可以限制内存的使用量
    ```sh
    zeyu@zeyu-desktop:~$ cat /proc/cgroups 
    #subsys_name	hierarchy	num_cgroups	enabled
    cpuset	9	1	1                   # 给cgroup中的任务分配独立的CPU(多处理器系统) 和内存节点
    cpu	8	102	1                       # 限制CPU时间片的分配，与cpuacct挂载在同一目录
    cpuacct	8	102	1                   # 生成cgroup中的任务占用CPU资源的报告，与cpu挂载在同一目录 
    blkio	11	102	1                   # 对块设备的IO进行限制
    memory	7	166	1                   # 对cgroup中的任务的可用内存进行限制，并自动生成资源占用报告
    devices	12	102	1                   # 允许或禁止cgroup中的任务访问设备
    freezer	4	2	1                   # 暂停/恢复cgroup中的任务
    net_cls	2	1	1                   # 使用等级识别符（classid）标记网络数据包，这让Linux流量控制器(Traffic Controller,TC)可以识别来自特定cgroup任务的数据包，并进行网络限制
    perf_event	3	1	1               # 允许使用perf工具来监控 cgroup
    net_prio	2	1	1               # 允许基于cgroup设置网络流量(netowork traffic)的优先级
    hugetlb	5	1	1                   # 限制使用的内存页数量
    pids	10	105	1                   # 限制任务的数量
    rdma	6	1	1                   # 限制RDMA/IB-specific的使用数量
    ```
* **Hierarchy**(层级)：层级由一系列`cgroup`以一个树状结构排列而成，每个层级通过绑定对应的子系统进行资源控制。层级中的`cgroup `节点可以包含零个或多个子节点，子节点继承父节点挂载的子系统。一个操作系统中可以有多个层级。系统中的多个`cgroup`构成的层级并非单根结构，可以允许存在多个,如果只有一个层级，那么所有的任务都将被迫绑定其上的所有子系统，这会给某些任务造成不必要的限制

### 规则
* 同一个层级可以附加一个或多个子系统
* 一个子系统可以附加到多个层级，当且仅当目标层级只有唯一一个子系统时(一个已经附加在某个层级上的子系统不能附加到其他含有别的子系统的层级上)
* 系统每次新建一个层级时，该系统上的所有任务默认加入这个新建层级的初始化`cgroup`，这个`cgroup`也被称为`root cgroup`
* 对于创建的每个层级，任务只能存在于其中一个`cgroup`中，即一个任务不能存在于同一个层级的不同`cgroup`中，但一个任务可以存在于不同层级中的多个`cgroup`中。如果操作时把一个任务添加到同一个层级中的另一个`cgroup`中，则会将它从第一个`cgroup`中移除
* 任务在`fork/clone`自身时创建的子任务默认与原任务在同一个`cgroup`中，但是子任务允许被移动到不同的`cgroup`中 
* 如果新建的层级结构要绑定的子系统与目前已经存在的层级结构完全相同，那么新的挂载会重用原来已经存在的那一套
* 目前无法将一个新的子系统绑定到激活的层级上，或者从一个激活的层级中解除某个子系统的绑定
* 创建的层级中创建文件夹，就类似于`fork`了一个后代`cgroup`，后代`cgroup`中默认继承原有`cgroup`中的配置属性，但是可以根据需求对配置参数进行调整
* 一个顶层的`cgroup`文件系统被卸载（unmount）时，如果其中创建过深层次的后代`cgroup`目录，那么就算上层的`cgroup`被卸载了，层级也是激活状态，其后代`cgroup`中的配置依旧有效。只有递归式地卸载层级中的所有`cgroup`，那个层级才会被真正删除

### cgroup中文件
- **tasks**：这个文件中罗列了所有在该`cgroup`中任务的`TID`，即线程`ID`(gettid)
- **cgroup.procs**：这个文件罗列所有在该`cgroup`中的`TGID`（线程组ID），即线程组中第一个进程的`PID`
- **notify_on_release**：填0或1，表示是否在`cgroup`中最后一个任务退出时通知运行`release agent`，默认情况下是0，表示不运行
- **release_agent**：指定`release agent`执行脚本的文件路径（该文件在最顶层`cgroup`目录中存在），这个脚本通常用于自动化卸载无用的`cgroup`

###  /proc/[pid]/cgroup

```sh
zeyu@zeyu-desktop:~/work$ cat /proc/self/cgroup 
12:devices:/user.slice
11:blkio:/user.slice
10:pids:/user.slice/user-1001.slice/user@1001.service
9:cpuset:/
8:cpu,cpuacct:/user.slice
7:memory:/user.slice/user-1001.slice/user@1001.service
6:rdma:/
5:hugetlb:/
4:freezer:/
3:perf_event:/
2:net_cls,net_prio:/
1:name=systemd:/user.slice/user-1001.slice/user@1001.service/apps.slice/apps-org.gnome.Terminal.slice/vte-spawn-b3ed7abd-c7b2-4837-a6b7-9bbdce65c6bc.scope
0::/user.slice/user-1001.slice/user@1001.service/apps.slice/apps-org.gnome.Terminal.slice/vte-spawn-b3ed7abd-c7b2-4837-a6b7-9bbdce65c6bc.scope
```

每一行包含用冒号隔开的三列，他们的含义分别是：

- `cgroup`树的`ID`， 和`/proc/cgroups`文件中的`ID`一一对应
- 和`cgroup`树绑定的所有`subsystem`，多个`subsystem`之间用逗号隔开。这里`name=systemd`表示没有和任何`subsystem`绑定，只是给他起了个名字叫`systemd`
- 进程在`cgroup`树中的路径，即进程所属的`cgroup`，这个路径是相对于挂载点的相对路径

## 命令
- 创建层级
    ```sh
    sudo mount -t cgroup -o cpuset cgroup /cgroup 
    ```
- 相关工具
    ```sh
    # 安装cgroup工具
    apt install cgroup-tools
    # 创建子控制组群 (mkdir)
    sudo cgcreate -g cpuset:test
    # 设置cpu子系统参数 (echo >)
    sudo cgset -r cpuset.cpus=1 test/
    # 在cgroup中执行命令
    sudo cgexec -g memory:test ls
    # 将进程加入cpu子控制群组 (echo >>)
    sudo cgclassify -g memory:test 133719
    # 删除cpu子控制组 (rmdir)
    sudo cgdelete memory:test
    # 查看所有的cgroup
    lscgroup
    # 查看所有支持的子系统
    lssubsys -a
    # 查看所有子系统挂载的位置
    lssubsys -m
    # 查看单个子系统（如memory）的挂载位置
    lssubsys -m memory
    ```

## cgroups v1 & cgroups v2
尽管`cgroups v2`(Linux 4.5)旨在取代`cgroups v1`(Linux 2.6.24)，但出于兼容性原因，不太可能完全取代。目前，`cgroups v2`只实现`cgroups v1`中可用的控制器的子集

`cgroups v1`和`cgroups v2`可以互补使用,但是一个子系统不能同时用于`cgroups v1`层次结构和`cgroups v2`层次结构

## cgroups v1
在`cgroups v1`下，每个控制器可以安装在一个单独的`cgroup`文件系统上，该文件系统为系统上的进程提供了自己的层次结构,也可以多个控制器一起安装在同一个`cgroup`文件系统。在已经挂载的层次结构中,每一个目录为一个控制组,子目录为子控制组。

在`cgroups v1`中可以区分进程和线程,但对于内存控制器来说毫无意义，因为进程的所有线程共享一个地址空间，对于控制进程中线程这一问题在`cgroup v2`中被调整

`cgroups`使用需要开启内核编译选项`CONFIG_CGROUP`,此外，每个`v1`控制器都有一个相关的配置选项，必须设置该选项才能使用该控制器

为了使用`v1`控制器，它必须安装在`cgroup`文件系统上。此类安装的通常位置在`tmpfs`文件系统(/sys/fs/cgroup)
    ```sh
        mount -t cgroup -o cpu none /sys/fs/cgroup/cpu
    ```
可以针对同一层次结构同时安装多个控制器
    ```sh
        mount -t cgroup -o cpu,cpuacct none /sys/fs/cgroup/cpu,cpuacct
        mount -t cgroup -o all cgroup /sys/fs/cgroup
    ```
安装没有附加控制器的`cgroup`层次结构,这种层次结构的唯一目的是跟踪进程
    ```sh
        mount -t cgroup -o none,name=somename none /some/mount/point
    ```
通过写入`cgroup.procs`将对应进程以及其线程加入一个`cgroup`，一次只能将一个`PID`写入此文件，写入`0`将导致写入的进程加入对应`cgroup`




