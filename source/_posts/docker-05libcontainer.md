---
title: Docker - 05 libcontainer
date: 2022-10-31 09:00:00
tags:
    - Docker
---

`libcontainer`已经被`docker`捐给`OCI`,并包装成`runc`:
[runc](https://github.com/opencontainers/runc)

以下内容基于`runc 1.0.3`版本展开

## OCI Bundle
`OCI Bundle`是指满足`OCI`标准的一系列文件，这些文件包含了运行容器所需要的所有数据，它们存放在一个共同的目录，该目录包含以下两项：
- `config.json`文件包含容器运行的配置数据
- `root`目录为`container`的`root filesystem`

## 整理流程
在`runc`中所有二级命令都有单独的`go`文件，通过`packege cli`中的`App`命令模版统一调用


## 创建容器
`runc create -b /mycontainer test`

- `create.go`
    ```go
	    Action: func(context *cli.Context) error {
            // 入参检查
		    if err := checkArgs(context, 1, exactArgs); err != nil {
			    return err
		    }
            //保存pid文件
		    if err := revisePidFile(context); err != nil {
			    return err
		    }
            //加载config.json到spec数据结构
		    spec, err := setupSpec(context)
		    if err != nil {
			    return err
		    }
            //创建容器
		    status, err := startContainer(context, spec, CT_ACT_CREATE, nil)
		    if err != nil {
			    return err
		    }
		    // exit with the container's exit status so any external supervisor is
		    // notified of the exit with the correct exit status.
		    os.Exit(status)
		    return nil
	    },
    ```
- `utils_linux.go/createContainer/CreateLibcontainerConfig`将`Spec`等相关配置信息转存到`Config`数据结构,为后续工厂执行操作提供入参
- 加载`Factory`接口，其中`LinuxFactory`将作为一种实现
    - 创建`root`目录 `root`用户执行是`/run/runc`
    - 创建`LinuxFactory`数据结构
- 调用`Factory`的`Create()`方法，返回`Container`接口**创建逻辑容器**,其中`linuxContainer`将作为其中一种实现
    - 验证容器`id`合法性
    - 验证`config`合法性
    - 通过`SecureJoinVFS`创建容器的`root`目录 `${root}/containerid`
    - 返回`Container`对象
- 创建`runner`在`r.run(spec.Process)`中执行`container.Start(process)`**创建物理容器**`Container`主要包含了容器配置、控制等信息，是对不同操作系统下容器实现的抽象,目前在`linux`平台相当于执行`func (c *linuxContainer) Start(process *Process) error`
    - 在容器`root`目录下创建`exec.fifo`
    - 执行内置函数`func (c *linuxContainer) start(process *Process) (retErr error)`
        - 创建`parentProcess`,参与物理容器创建过程的`Process`一共有两个实例
            - 第一个叫`Process`，用于物理容器内进程的配置和`IO`的管理，前面在`runner run`创建的`Process`就是指它,该实例主要来自`spec.Process`
            - 另一个叫`ParentProcess`，负责从物理容器外部处理物理容器启动工作，与`Container`对象直接进行交互。启动工作完成后，`ParentProcess`负责执行等待、发信号、获得容器内进程`pid`等管理工作,`initProcess`是`ParentProcess`的一个实现

                ```go
                type parentProcess interface {
            	    // pid returns the pid for the running process.
            	    pid() int

            	    // start starts the process execution.
            	    start() error

            	    // send a SIGKILL to the process and wait for the exit.
            	    terminate() error

            	    // wait waits on the process returning the process state.
            	    wait() (*os.ProcessState, error)

            	    // startTime returns the process start time.
            	    startTime() (uint64, error)

            	    signal(os.Signal) error

            	    externalDescriptors() []string

            	    setExternalDescriptors(fds []string)

            	    forwardChildLogs() chan error
                }

                type initProcess struct {
	                cmd             *exec.Cmd
	                messageSockPair filePair
	                logFilePair     filePair
	                config          *initConfig
	                manager         cgroups.Manager
	                intelRdtManager intelrdt.Manager
	                container       *linuxContainer
	                fds             []string
	                process         *Process
	                bootstrapData   io.Reader
	                sharePidns      bool
                }
                ```
        - 调用`parentProcess`的`forwardChildLogs`通过管道不断获取容器内的日志
        - 调用`parentProcess`的`start`，其实就是执行`func (p *initProcess) start() (retErr error)`
            - 这里会执行`runc init`,创建子进程完成容器的初始化工作,父子进程通过管道通信


