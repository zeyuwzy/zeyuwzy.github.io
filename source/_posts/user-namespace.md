---
title: User Namespace
date: 2022-07-14 08:49:11
tags:
    - Linux
    - Linux Namespace
---

## 描述
用户命名空间隔离了安全相关的标识和属性,尤其是`uid`和`gid`(credentials)、根目录、密钥(keyrings)、能力(capabilities)。一个进程的`uid`,`gid`在不同的用户命名空间可能是不同的。一个进程在在一个用户命名空间外可能是一个普通的非特权用户,同时在另一个用户命名空间内可能是uid = 0的root。换句话说,这个进程对一个用户命名空间内的操作具有完全的权限,但是对用户命名空间外的操作没有权限

## 嵌套用户命名空间
用户名称空间可以嵌套,也就是说除了初始根用户命名空间之外,每个用户命名空间都有一个父用户命名空间,并且可以具有零个或多个子用户命名空间,父用户命名空间通过`unshare`、`clone`,配合使用`CLONE_NEWUSER`标志创建子用户命名空间。内核（>3.11)限制用户命名空间最多嵌套32层,超出限制会报`EUSERS`

每一个进程都是一个用户命名空间的成员,一个进程通过`fork`或者`clone`但是不使用`CLONE_NEWUSER`就会和父进程一样作为同一个用户命名空间的成员。一个进程可以通过`setns`加入另一个用户命名空间,前提是这个进程在这个用户命名空间有`CAP_SYS_ADMIN`能力,切换后这个进程将在这个用户命名空间有全部能力

ioctl中的`NS_GET_PARENT`用于获取用户命名空间的父子关系(ioctl_ns)

## 用户命名空间能力
通过`clone`或者`unshare`创建一个新的用户命名空间,或者通过`setns`加入一个已经存在的用户命名空间都会使这个进程获得这个用户命名空间的全部能力集,并且这个进程在父用户命名空间或切换之前的用户命名空间没有任何能力,即使新的用户命名空间是由root创建或切换的

在用户命名空间中调用`execve`通常将以正常的方式重新计算新进程的能力,因此除非uid=0或者文件的`inheritable capabilities mask`不为空,能力都将被丢掉

`clone`,`unshare`,`setns`创建/切换用户命名空间都将导致`securebits`标志位变成默认值(全关),因为切换到新的用户命名空间的进程在原用户命名空间将失去所有能力,所以即使将`securebits`标志位恢复,也没有办法通过两次`setns`在两个用户命名空间中切换。

判断进程在用户命名空间能力规则如下

1. 如果进程是该用户命名空间的成员,并且`effective set`有能力,那么该进程在这个用户命名空间就有能力。通过执行`set-user-ID`或者有文件能力的可执行文件都可以让新进程获得能力,或者`clone`,`unshare`,`setns`创建新的用户命名空间
2. 如果进程在一个用户命名空间中有能力,那么它在所有子用户命名空间都有对应能力
3. 当进程创建用户命名空间时,内核将记录进程的`euid`作为该用户命名空间的拥有者,处于父用户命名空间并且`euid`与用户命名空间拥有者相匹配的进程将获得该用户命名空间以及将来子用户命名空间的全部能力,`ioctl`中`NS_GET_OWNER_UID`用于获取用户命名空间的拥有者的uid

## 用户命名空间内能力的影响
在一个用户命名空间拥有能力意味着该进程可以对该用户命名空间所持有的其他命名空间的资源执行特权操作

还有一些特权操作影响的资源不属于任何命名空间,例如：修改系统时间(CAP_SYS_TIME), 加载内核模块(CAP_SYS_MODULE),创建一个设备(CAP_MKNOD),只有初始用户命名空间的特权进程有能力执行上述操作

在拥有`mount namespace`的用户命名空间内拥有`CAP_SYS_ADMIN`的进程允许在创建绑定挂载或者挂载以下文件系统
   * /proc (since Linux 3.8)
   * /sys (since Linux 3.8)
   * devpts (since Linux 3.9)
   * tmpfs(5) (since Linux 3.9)
   * ramfs (since Linux 3.9)
   * mqueue (since Linux 3.9)
   * bpf (since Linux 4.4)

在拥有`cgroup namespace`的用户命名空间内拥有`CAP_SYS_ADMIN`的进程允许挂载`cgroup version 2` 文件系统和`cgroup version 1 named hierarchies` (cgroup filesystems mounted with the "none,name=" option)

在拥有`pid namespace`的用户命名空间内拥有`CAP_SYS_ADMIN`的进程允许挂载`/proc`

但是要挂载基于块的文件系统只能是在初始用户命名空间拥有`CAP_SYS_ADMIN`的进程

## 用户命名空间和其他命名空间交互
从Linux3.8开始,非特权进程可以创建用户命名空间,其他类型的名称空间只需基于调用的用户命名空间中有`CAP_SYS_ADMIN`能力就可以创建

创建其他命名空间时,它由创建进程所对应的用户命名空间所有。对于其他命名空间的特权操作要求进程具有必要的用户命名空间中的能力

调用`clone`,`unshare`,如果`CLONE_NEWUSER`和其他`CLONE_*`一起声明,那么会保证首先创建用户命名空间,给其他命名空间的创建提供权限。这种参数的组合方式可以被非特权进程调用

当一个新的命名空间创建(用户命名空间除外),内核都会记录这个进程的用户命名空间作为新创建的命名空间的拥有者(这种关联不能被改变),当一个进程要特权操作一个命名空间隔离的资源时,内核就会检查这个进程在要操作的那个命名空间对应用户命名空间有没有相关能力。例如,一个进程想要修改主机名(sethostname),该资源由`uts namespace`管理,这是内核就会检查哪一个用户命名空间拥有当前要操作的`uts namespace`,并检查该进程在那个用户命名空间是否有对应的能力`CAP_SYS_ADMIN`

NS_GET_USERNS ioctl(2) 用于获取用户命名空间拥有哪些其他命名空间(ioctl_ns)

## uid_map gid_map
当一个用户命名空间被创建时,一开始并没有将`uid`, `gid`映射到父命名空间,`/proc/[pid]/uid_map`,`/proc/[pid]/gid_map`用于将命名空间内的`uid`,`gid`映射。这些文件只能被写入一次

映射文件映射了pid的进程和打开map文件的进程的uid/gid,不同用户命名空间下的进程打开同一个map文件可能看到不同的结果

这个文件在用户命名空间创建之初是空的,映射文件中的每一行指定两个用户名称空间之间一系列连续用户ID的1:1映射,每行中的规范采用由空格分隔的三个数字的形式。前两个数字分别指定两个用户命名空间中的起始用户ID,第三个数字指定映射的长度

第一个字段描述了pid所在用户命名空间id映射的开始位置,如果这两个进程(pid和打开map的进程)位于不同的用户命名空间中,字段2是打开map进程对应的用户命名空间对应id范围的开始位置,如果这两个进程位于同一个用户命名空间,字段2是该用户命名空间的父命名空间对应id范围的开始位置,一般打开`/proc/self/uid_map`就是查看当前进程的用户命名空间和父用户命名空间的id映射关系

`getuid`,`getgid`,`stat`等用于身份认证的系统调用都将返回映射到调用用户命名空间的id

当进程访问文件时,其用户和组ID映射到初始用户命名空间中,以进行权限检查和在创建文件时分配ID,当进程调用`stat`获取文件的uid,gid时,这些ID又会以相反的方向映射回对应的命名空间

初始用户名称空间没有父名称空间,但为了一致性,内核为以下对象提供了虚拟用户和组ID映射文件：
    cat /proc/$$/uid_map
    0          0 4294967295
    (2^32 - 1)为未映射,类似(uid_t) -1没有这个用户

## 定义uid_map gid_map
在创建用户命名空间后,可以将这个用户命名空间中的某个进程的map文件写入一次,去定义id的映射,尝试多次写入会导致`EPERM`

### 写入规则
* 三个字段必须都是有效值,而且最后一个字短必须大于0
* 每一行都以换行符终止
* 4.14内核限制写入5行,4.15之后限制写入340行,写入的字节数必须小于系统页面,写入必须在开头,不能用leesk,pwrite偏移写文件
* 3.8之前的内核要求映射的范围不能有重叠,连续行的字段1和字段2必须以升序数字顺序排列,3.9取消了这个限制
* 最少一行要被写入文件

违反上述规则的写入失败,错误为`EINVAL`  

### 针对进程的写入条件

1. 写入的进行必须要有对应pid进程所在用户命名空间的`CAP_SETUID`,`CAP_SETGID`
2. 写入进程必须位于进程pid的用户命名空间中,或位于进程pid的父用户命名空间中
3. 映射的ID必须在父命名空间也有映射关系
4. 以下两条满足其一:
   1. 写入进程在父用户命名空间中都具有`CAP_SETUID`,`CAP_SETGID`功能, 没有其他限制:该进程可以映射到父用户命名空间中的任意uid(gid)
   2. 否则,以下所有限制均适用: 
       * 写入的数据必须由单行组成,映射写入进程在父用户命名空间的euid和要映射的命名空间的uid
       * 写入进程必须与创建用户命名空间的进程有相同的euid
       * 在gid映射的情况下,必须首先通过向`/proc/[pid]/setgroups`写入`deny`来拒绝使用`setgroups`系统调用

违反上述规则的写入将失败,并返回错误`EPERM`

## 对修改uid gid系统调用的影响
如果map文件没有,那么修改uid,gid会失败,否则,只有映射的id可以被修改

user IDs 系统调用: `setuid`, `setfsuid`,`setreuid`,`setresuid`. group IDs 系统调用:`setgid`,`setfsgid`,`setregid`, `setresgid`,`setgroups`

在写`/proc/[pid]/uid_map`之前在`/proc/[pid]/setgroups`写入`deny`将永久禁用用户命名空间的`setgroups`,并且允许在父用户命名空间没有`CAP_SETGID`能力的进程写`/proc/[pid]/gid_map`

## /proc/[pid]/setgroups
如果这个文件的值是`allow`,那么pid对应用户命名空间的进程有能力执行`setgroups`,如果文件的值是`deny`则相反。注意,如果没有写`/proc/[pid]/gid_map`那么无论`/proc/[pid]/setgroups`值是多少,进程有没有能力都无法执行`setgroups`

在写入`/proc/[pid]/gid_map`之前,pid对应用户命名空间的特权进程(CAP_SYS_ADMIN)有权限在这个文件写入`allow`或者`deny`

这个文件的默认值是`allow`

一旦`/proc/[pid]/gid_map`被写入,同时,setgroups的值是`allow`,那么就无法通过写入`deny`去关闭`setgroups`(EPERM)

一旦`setgroups`的值是`deny`,那么就无法通过写入`allow`去开启`setgroups`(EPERM)

限制的本质是`/proc/[pid]/gid_map`是否设置了(没设置就是`deny`),也就是说进程的`setgroups`权限不能从`allow` -> `deny`,只能从`deny` -> `allow`

子用户命名空间从其父命名空间继承`/proc/[pid]/setgroups`设置

`/proc/[pid]/setgroups`是3.19引入的内核,但是已经被移植到很多早期的版本中。因为它解决了涉及具有`rwx---rwx`等权限文件的访问权限问题,这些文件给所属组的权限比其他人要少,着就意味着一旦通过`setgroups`将进程的组都丢掉就可以让进程访问到它本来无法访问的文件。对于早期的用户命名空间来说这不是个问题,因为只有特权进程`CAP_SETGID`有能力调用`setgroups`,但是引入非特权用户创建用户命名空间后,就会导致本来没有特权的用户通过子命名空间获取到了能力,然后通过`setgroups`让进程访问到了本来没有权限访问的文件,`/proc/[pid]/setgroups`很好的通过拒绝任何非特权用户(CAP_SETGID)的路径使用`setgroup`

## 未映射的uid gid

在用户命名空间中有可以遇到未映射的id,比如在还没有写入`uid_map`之前调用`getuid`,这时会返回`overflow user ID`,默认值是65534(`/proc/sys/kernel/overflowuid`)

未映射的uid gid未转换为相应的溢出ID值。查看时uid/gid map,其中第二个字段没有映射,该字段显示为4294967295

影响的系统调用  
`The cases where unmapped IDs are mapped in this fashion include system calls that return user IDs (getuid(2), getgid(2), and similar), cre‐
       dentials passed over a UNIX domain socket, credentials returned by stat(2), waitid(2), and the System V IPC "ctl" IPC_STAT operations, cre‐
       dentials  exposed  by  /proc/[pid]/status  and the files in /proc/sysvipc/*, credentials returned via the si_uid field in the siginfo_t re‐
       ceived with a signal (see sigaction(2)), credentials written to the process accounting file (see acct(5)), and  credentials  returned  with
       POSIX message queue notifications (see mq_notify(3)).`

## 文件访问
为了在非特权进程访问文件时确定权限,进程凭UID、GID和文件的mode属性都映射回根用户命名空间中,然后进行比较以确定进程对文件的权限,其他类似身份认证+权限掩码的权限判断方式也是如此,比如System V IPC

## 文件相关操作的能力
有些能力允许进程在对不属于自己的文件操作时通过内核的限制: `CAP_CHOWN`,`CAP_DAC_OVERRIDE`,`CAP_DAC_READ_SEARCH`,`CAP_FOWNER`,`CAP_FSETID`

在一个用户命名空间中要生效需要满足
* 进程的`Effective set`在用户命名空间中有上述对应能力
* 文件的uid,gid都在用户命名空间中被映射(CAP_FOWNER只需要映射uid)

## Set-user-ID set-group-ID 程序
当在用户命名空间执行了一个`set-user-id`或者`set-group-id`的程序时,如果这个id在用户命名空间中没有被映射,那么新的程序被执行时就会忽略set-user-ID set-group-ID标志位,而使用原来的euid egid

## 其他
在用户命名空间中,不可能获得比的根用户更多的能力  
内核选项:CONFIG_USER_NS