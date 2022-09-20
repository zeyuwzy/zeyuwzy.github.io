---
title: Linux Command
date: 2021-02-27 10:00:00
tags:
    - Linux
---

## 系统目录结构
- bin:存放二进制可执行文件,date
- boot:存放开机启动程序
- dev:存放设备文件(device):字符设备、块设备,linux中所见皆文件
- home:存放普通用户
- etc:用户信息和系统配置文件passwd、group
- lib:库文件:libc.so.6
- root:管理员宿主目录（家目录）
- usr:用户资源管理目录 
- media:Linux识别光驱,U盘自动挂载   
- mnt:用户手动挂载

## 用户目录
- `$` 普通用户
- `#` 超级用户
- `cd -` 回到上一次的工作目录
- `.` 当前目录
- `..` 上一级目录
- `ls -a` 打开隐藏文件
- `ls -l` 文件详细信息

## 系统文件类型：
- 普通文件：-
- 目录文件：d
- 字符设备文件：c
- 块设备文件：b
- 软连接：l
- 管道文件：p
- 套接字：s
- 未知文件

## 命令
- which	    查看命令所在路径
- touch	    创建文件
- rm		删除目录/文件 
- rm -r 	递归删除
- cp -r		拷贝目录/文件
- cat		查看文件内容
- more less 查看大文件
- head tail 查看头尾
- tac		逆序查看文件内容
- more	    分屏查看内容
- mkdir -p  创建多层目录
- du -h     文件夹大小
- df -h     查看磁盘

## 软连接（快捷方式）
ln -s file file.soft (创建时务必对源文件使用 绝对路径，可以搬运)
## 硬链接
ln file file.hard

操作系统给每一个文件赋予唯一的inode,当有相同inode的文件存在时,彼此同步;删除时,只将硬链接计数减一。减为0时,inode被释放

## 文件属性和用户组
创建用户：

	sudo adduser 新用户名		--- useradd

修改文件所属用户：

	sudo chown 新用户名 待修改文件。

	sudo chown wangwu a.c

删除用户：

	sudo deluser 用户名  (-r 删除用户目录)

创建用户组：

	sudo addgroup 新组名

修改文件所属用户组：

	sudo chgrp 新用户组名 待修改文件。

	sudo chgrp g88 a.c

 删除组：

	sudo delgroup 用户组名 


使用chown 一次修改所有者和所属组：

	sudo chown 所有者：所属组  待操作文件。

    chmod u(user)/ g(group) /o(others)/a(all) + r/w/x  文件

## 查找与检索 

find
	-type 按文件类型搜索  d/p/s/c/b/l/ f(文件)

	-name 按文件名搜索

		find ./ -name "*file*.jpg"

	-maxdepth 指定搜索深度。应作为第一个参数出现。

		find ./ -maxdepth 1 -name "*file*.jpg"


	-size 按文件大小搜索. 单位：k、M、G

		find /home/itcast -size +20M -size -50M

	-atime、mtime、ctime 天  amin、mmin、cmin 分钟。

	-exec：将find搜索的结果集执行某一指定命令。

		find /usr/ -name "*tmp*" -exec ls -ld {} \;

	-ok: 以交互式的方式 将find搜索的结果集执行某一指定命令


	-xargs：将find搜索的结果集执行某一指定命令。  当结果集数量过大时，可以分片映射。

		find /usr/ -name "*tmp*" | xargs ls -ld 

	-print0：
		find /usr/ -name "*tmp*" -print0 | xargs  -0 ls -ld 

grep(按内容查找)

	grep -r "copy" ./ -n

	-n参数：:显示行号    -r ：递归

检索进程结果集

	ps aux | grep 'cupsd'  

## 安装与卸载
软件安装：

	1. 联网

	2. 更新软件资源列表到本地。  sudo apt-get update

	3. 安装 sudo apt-get install 软件名

	4. 卸载	sudo apt-get remove 软件名

	5. 使用软件包（.deb） 安装：	sudo dpkg -i 安装包名。

## 压缩解压
tar

    c 创建 -- 压缩
    x 释放 -- 解压
    v 显示提示信息
    f 指定压缩文件名
    z 使用gzip压缩
    j 使用bizp2方式压缩

tar压缩:

	tar zcvf  test.tar.gz  file1 dir2   使用 gzip方式压缩。

	tar jcvf  test.tar.gz  file1 dir2   使用 bzip方式压缩。

tar解压：

	将压缩命令中的 c --> x

		tar zxvf  test.tar.gz   使用 gzip方式解压缩。

		tar jxvf  test.tar.gz   使用 bzip2方式解压缩。

rar压缩：

	rar a  压缩包名（带.rar后缀） 压缩材料。

	rar a   testrar.rar	stdio.h test2.mp3

rar解压：

	rar x 压缩包名（带.rar后缀）

zip压缩：

	zip -r 压缩包名（带.zip后缀） 压缩材料。

	zip -r testzip.zip dir stdio.h test2.mp3

zip解压：

	unzip 压缩包名（带.zip后缀） 

	unzip  testzip.zip 