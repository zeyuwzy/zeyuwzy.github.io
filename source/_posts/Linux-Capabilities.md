---
title: Linux Capabilities
date: 2022-06-11 23:19:11
tags:
    - Linux
---

## 描述
为了执行权限检查，传统的UNIX实现区分了两类进程：特权进程(`euid=0`，称为超级用户或`root`)和非特权进程(`euid!=0`),特权进程绕过所有内核权限检查，而非特权进程则根据进程身份认证进行权限检查(通常为:`euid,egid,group list`)。从内核2.2开始，Linux将传统上与超级用户相关的权限划分为不同的单元——称为能力。能力可以单独启用和禁用,是每个线程的属性 

## 能力的要求
1. 对于所有特权操作，内核必须检查对应线程`effective set`是否有该能力
2. 内核必须提供允许更改和检索线程能力集的系统调用 
3. 文件系统必须支持将能力附加到可执行文件,以便进程在文件被执行时获得这些能力

## 针对内核开发者
当为内核开发新功能时，这个功能需要被能力管控

1. 能力目标是将超级用户的能力分成几部分，这样，如果一个程序具有一个或多个能力,它对系统造成破坏的能力将小于使用root权限运行的同一程序
2. 您可以选择为新功能创建新能力，或将该功能与现有能力关联,为了将能力集保持在一个可管理的规模，除非有令人信服的选择前者的原因。(还有一个技术限制：能力集的大小目前限制为64位)
3. 要确定哪个现有功能最适合与您的新功能相关联，请查看上面的功能列表，以便找到最适合您的新功能的能力。一种方法是确定是否有其他功能需要能力并且将始终与新功能一起使用的可能性。
4. 尽可能不要为新功能使用`CAP_SYS_ADMIN`能力,这个能力权限太高(被称为 the new root)
5. 如果您确定确实有必要为您的功能创建新能力，请不要将其命名为“一次性使用”,要让这个能力的名称可以适用于更多其他的功能

## 线程能力
每个线程都有以下能力集，其中包含零个或多个上述能力
- Permitted  
用于限制`Effective set`的超集,同时也可以限制想添加能力到`Inheritable set`但是 `Effective set`没有`CAP_SETPCAP`能力的情况  
如果一个线程从`Permitted set`丢掉一个能力，那么它就无法再获取这个能力(除非它`execve`一个suid=0程序或者一个程序的相关可执行文件有对应的能力)

- Inheritable  
这个集合声明`execve`继承的能力,当程序运行时，要执行的文件的`file inheritable set`和当前程序 `Inheritable set`相符合时，就会被添加到新的程序`Permitted set`

- Effective  
这是内核用来对线程执行权限检查的一组能力

- Bounding (2.6.25后变成线程级)  
可用于限制在执行`execve`时新进程获得的能力

- Ambient (4.3后内核)  
该能力主要针对非特权用户(`prctl`可直接修改能力),非特权程序`A`调用`execv`系列函数执行新的程序`B`时，程序`A`的`ambient`权限集就会被进程`B`所继承

  1. 如果`Permitted`或者`Inheritable`中的某个能力`P`被清空，在`Ambient`中该能力`P`也自动被清空
  2. 当程序调用`setuid`改变`UID`或者`setgid`改变`GID`时，`Ambient`中的所有能力会被清空
  3. 调用`execve`函数执行一个程序时，如果该文件曾经被设置了任何的文件`capabilities`，`Ambient`中的所有能力会被清空；
  4. 当调用`execve`函数执行一个程序时，如果该文件没有设置过任何的文件`capabilities`，`Ambient`中的所有能力就会被赋予给新程序的`Permited`和`Effective`能力集。

## 文件能力
可执行文件的能力会被存储在文件扩展属性中(`security.capability`),可以通过`setcap`, `cap_set_file`,`cap_set_fd`,设置文件的能力。对文件写入扩展属性这一过程需要`CAP_SETFCAP`能力。文件的能力将与线程的能力一起决定`execve`后新进程的能力

- Permitted  
无论线程的`Inheritable`能力集如何，文件`Permitted`的能力都会与线程的`Bounding`集合与运算，添加到新线程的`Permitted`

- Inheritable  
文件的`Inheritable`会和线程的`Inheritable`做与运算,添加到`execv`后新进程的`Permitted`

- Effective  
这不是一个集合，仅仅是一个标志位。如果设置开启，那么在执行完`execve`后，线程`Permitted`集合中的`capabilities`会自动添加到它的`Effective`集合中。对于一些旧的可执行文件，由于其不会调用 `capabilities`相关函数设置自身的`Effective`集合，所以可以将可执行文件的`Effective bit`开启，从而可以将`Permitted`集合中的 `capabilities`自动添加到`Effective`集合中

### 文件扩展属性
`VFS_CAP_REVISION_3`和命名空间有关，当线程处于一个子命名空间中，并且这个线程在该命名空间有`CAP_SETFCAP`能力，那么这个线程为文件添加能力时文件security.capability就是`VFS_CAP_REVISION_3`版本，与`VFS_CAP_REVISION_2`的区别体现在扩展属性会将当前命名空间的root user id一起编码加到扩展属性中

## 能力继承

### fork
通过`fork`创建的子进程继承其父进程的全部能力集

### execve
P' : 新进程
P  : 旧进程
F  : 文件


- P'(ambient) = (file is privileged) ? 0 : P(ambient)
- P'(permitted) = (P(inheritable) & F(inheritable)) | (F(permitted) & P(bounding)) | P'(ambient)
- P'(effective) = F(effective) ? P'(permitted) : P'(ambient)
- P'(inheritable) = P(inheritable)
- P'(bounding) = P(bounding) 

普通用户执行`execve`时`permitted`和`effective`都会被清空  
root用户执行`execve`时`ambient`会被清空

### old bin
对于老的二进制文件，没有使用libcap的api去获得能力，而是使用文件的能力，这就需要将文件的`effective set`开启，这样执行时会自动将文件的`permitted`添加到对应进程的`effective`

### root

1. 如果进程的real uid或euid = 0,那么`execve`文件的`inheritalbe`和`permitted`就会被忽略(通常意味着新进程所有权限都有,有一种特殊情况：一个普通进程`execve`一个`setuid root`的可执行文件,并且这个文件添加了额外的能力,这时新进程只会获得旧进程的能力，不会获取来自文件的能力,通过这种方式我们可以实现让一个进程切换到euid=0但是却没能力的状态)
2. 如果进程euid=0,那么`execve`文件的`effective bit`就会默认为开启状态

所以当一个普通程序execve一个setuid root的可执行文件（这个文件没添加额外的能力），或者一个real uid | euid = root程序执行execve时：

- P'(permitted) = P(inheritable) | P(bounding)
- P'(effective) = P'(permitted)


## bounding set

`bounding set`用于限制`execve`时新进程能力的一组能力集

- `bounding set`会和文件的`permitted set`做与运算后添加到新进程的`permitted set`
- 如果一个能力不在`bounding set`，那么也无法将这个能力添加到`inheritable set`,即使这个能力在`permitted set`

在计算`execve`新进程的`permitted set`时，`bounding set`只会和文件的`permitted set`与运算，不会与`inheritalbe set`与运算,如果一个能力在进程的`inheritable set`而不在`bounding set`时，只要文件的`inheritable set`有该能力，还是可以将这个能力添加到新进程`permitted set`
 
### 2.6.25后的内核
`bounding set`不再是系统范围的能力集，而是作用到每个线程

`bounding set`本身在`fork`和`execve`都是直接继承的

线程可以通过使用`prctl`的`PR_CAPBSET_DROP`去除一个`bounding set`能力，能力一旦被去除就不能恢复,可以使用`prctl`的`PR_CAPBSET_READ`去读取能力是否在`bounding set`

从`bounding set`中删除一个能力不会让`inheritable set`对应的能力删除，但是却能阻止向`inheritable set`添加对应能力


## EUID 改变时的能力

1. 如果`real`,`effective`或者`saved set user IDs`原先是0，通过`setuid`导致这些值变为非0，那么`permitted`,`effective`,和`ambient`集中所有能力都会被清除
2. 如果`euid`从0变为非0，那么`effective set`中所有能力都会清除
3. 如果`euid`从非0变为0，那么`permitted set`中所有能力都将拷贝到`effective set`
4. 如果`fsuid`从0变为非0，那么`effective set`中的`CAP_CHOWN, CAP_DAC_OVERRIDE, CAP_DAC_READ_SEARCH, CAP_FOWNER, CAP_FSETID, CAP_LINUX_IMMUTABLE (since Linux  2.6.30),  CAP_MAC_OVERRIDE, and CAP_MKNOD(since Linux 2.6.30)`会被清除,相反如果`fsuid`从非0变为0，那么任何一个上述能力只要在`permitted set`中都会被添加到`effective set`

如果一个uid=0 euid=0的线程在setuid到普通用户时想要保存之前的能力，可以使用`SECBIT_KEEP_CAPS`标志位


## 以编程的方式调整能力集

一个线程可以使用capget,capset系统调用去改变permitted, effective,和inheritable能力集

但是更推荐使用cap_get_proc,cap_set_proc去完成上述功能

1. 如果一个线程没有`CAP_SETPCAP`能力,那么新的`inheritable set`一定是现有`inheritable set ＋ permitted set`的子集
2. 新的`inheritable set`一定是现有`inheritable set + bounding set`的子集
3. 新的`permitted set`一定是现有`permitted set`的子集
4. 新的`effective set`一定是新的`permitted set`的子集


## 安全参数
自2.6.26，内核启用文件能力,linux声明了一系列线程级的安全位参数去禁用一些uid=0的线程对应的能力

- SECBIT_KEEP_CAPS  
  1. 设置了这个标志位可以在线程从UIDs=0 切换到非0时保存`permitted set`的能力，否则`permitted set`的能力都将清空  
  2. 这个标志位本身在execve时会被清空  
  3. 请注意，即使设置了`SECBIT_KEEP_CAPS`,`effective set`在从euid=0切换到非0时也会被清空,但是如果线程设置了这个标志位但是它的euid不等于0，那么他再切换到其他UIDs不等于0的线程就不会清空`effective set`  
  4. 如果设置了`SECBIT_NO_SETUID_FIXUP`,那么`SECBIT_KEEP_CAPS`就会被屏蔽  
  5. 这个标志位和旧版的prctl`PR_SET_KEEPCAPS`功能相同  


- SECBIT_NO_SETUID_FIXUP  
这个设置将在线程的euid fsuid从0和非0切换时阻止内核调整线程的permitted,effective,ambient集

- SECBIT_NOROOT  
如果设置了这个标志位，那么suid的可执行文件被执行或者euid ruid = 0执行一个程序时新程序不会获得root对应的能力

- SECBIT_NO_CAP_AMBIENT_RAISE  
设置这个标志位将不允许提升ambient set的能力通过prctl PR_CAP_AMBIENT_RAISE 选项

每一个标志都有一个`lock`标志,任何`lock`标志都是不可逆的，一旦设置锁定标志位，原来的基础标志位就不可更改,锁定标志位包含：SECBIT_KEEP_CAPS_LOCKED, SECBIT_NO_SETUID_FIXUP_LOCKED,
SECBIT_NOROOT_LOCKED,SECBIT_NO_CAP_AMBIENT_RAISE_LOCKED.

安全参数可以通过prctl的PR_SET_SECUREBITS 和 PR_GET_SECUREBITS去获取和更改, `CAP_SETPCAP`能力具有修改安全参数的功能,注意：只有包含<linux/securebits.h>才可以使用SECBIT*相关内容

安全标志位除了SECBIT_KEEP_CAPS始终会被清除,其余都会被子进程继承

使用以下标志位可以锁定自身以及子进程的能力，除非对应执行的文件有能力

```c
       prctl(PR_SET_SECUREBITS,
                   /* SECBIT_KEEP_CAPS off */
                   SECBIT_KEEP_CAPS_LOCKED |
                   SECBIT_NO_SETUID_FIXUP |
                   SECBIT_NO_SETUID_FIXUP_LOCKED |
                   SECBIT_NOROOT |
                   SECBIT_NOROOT_LOCKED);
                   /* Setting/locking SECBIT_NO_CAP_AMBIENT_RAISE
                      is not required */

```


## namespace
### suid root
当一个suid程序的uid匹配上了一个创建新namespace的uid时，当这个程序在命名空间内被任意程序执行时,就会被授予进程的permitted和effective集的能力,这里执行suid程序新进程获得的能力与上文执行suid程序获取能力的策略一致，只不过这里root程序是创建namespace对应的那个uid

### file cap

传统的文件能力只是与二进制文件的一系列能力集有关，并不关心namespace,后来引入`VFS_CAP_REVISION_3`,在传统文件能力后又添加了namespace的root uid,在命名空间执行一个程序时，只有当文件setuid的 uid与扩展属性的uid匹配上时才会授予相关权限 

## 其他
内核选项`CONFIG_SECURITY_CAPABILITIES`去开启,关闭能力

/proc/[pid]/task/TID/status 去看线程的能力

/proc/[pid]/status 去看进程的能力
