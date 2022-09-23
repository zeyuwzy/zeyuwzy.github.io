---
title: Linux dmesg
date: 2021-04-27 10:00:00
tags:
    - Linux
---

dmesg -L # color

dmesg -H # human timestamp

dmesg -T # readable timestamp

dmesg --follow # 持续观察输出

dmesg | tail -10 # 最后10行，当然也可以使用其它命令，如more，less，grep

dmesg -l info 输出info级别


dmesg根据用户类别对日志进行了分组

dmesg   -f(facility)

    kern: Kernel messages.
    user: User-level messages.
    mail: Mail system.
    daemon: System daemons.
    auth: Security/authorization messages.
    syslog: Internal syslogd messages.
    lpr: Line printer subsystem.
    news: Network news subsystem.
  
使用 -x(decode) 参数可以输出包括组和日志级别的信息。

清空dmesg缓冲区日志  dmesg -c

我们可以使用如下命令来清空dmesg的日志。该命令会清空dmesg环形缓冲区中的日志。但是你依然可以查看存储在`/var/log/dmesg`文件中的日志。你连接任何的设备都会产生dmesg日志输出。
