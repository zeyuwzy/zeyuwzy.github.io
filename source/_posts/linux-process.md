---
title: Linux Process
date: 2021-05-15 10:00:00
tags:
    - Linux
---

进程：

	程序：死的。只占用磁盘空间。		

	进程；活的。运行起来的程序。占用内存、cpu等系统资源。

PCB进程控制块：

	进程id

	文件描述符表

	进程状态：	初始态、就绪态、运行态、挂起态、终止态。

	进程工作目录位置

	*umask掩码 

	信号相关信息资源。

	用户id和组id

fork函数：

	pid_t fork(void)

	创建子进程。父子进程各自返回。父进程返回子进程pid。 子进程返回 0.

	getpid();getppid();

	循环创建N个子进程模型。 每个子进程标识自己的身份。

父子进程相同：

	刚fork后。 data段、text段、堆、栈、环境变量、全局变量、宿主目录位置、进程工作目录位置、信号处理方式


父子进程不同：

	进程id、返回值、各自的父进程、进程创建时间、闹钟、未决信号集

父子进程共享：

	读时共享、写时复制。———————— 全局变量。

	1. 文件描述符 2. mmap映射区。

	
	

gdb调试：

	设置父进程调试路径：set follow-fork-mode parent (默认)

	设置子进程调试路径：set follow-fork-mode child


exec函数族：

	使进程执行某一程序。成功无返回值，失败返回 -1

	int execlp(const char *file, const char *arg, ...);		借助 PATH 环境变量找寻待执行程序

		参1： 程序名

		参2： argv0

		参3： argv1

		...： argvN

		哨兵：NULL

	int execl(const char *path, const char *arg, ...);		自己指定待执行程序路径。

	int execvp();

ps ajx --> pid ppid gid sid 


孤儿进程：

	父进程先于子进终止，子进程沦为“孤儿进程”，会被 init 进程领养。

僵尸进程：

	子进程终止，父进程尚未对子进程进行回收，在此期间，子进程为“僵尸进程”。  kill 对其无效。


wait函数：	回收子进程退出资源， 阻塞回收任意一个。

	pid_t wait(int *status)

	参数：（传出） 回收进程的状态。

	返回值：成功： 回收进程的pid

		失败： -1， errno

	函数作用1：	         阻塞等待子进程退出

	函数作用2：	清理子进程残留在内核的 pcb 资源

	函数作用3：	通过传出参数，得到子进程结束状态

	
	获取子进程正常终止值：

		WIFEXITED(status) --》 为真 --》调用 WEXITSTATUS(status) --》 得到 子进程 退出值。

	获取导致子进程异常终止信号：

		WIFSIGNALED(status) --》 为真 --》调用 WTERMSIG(status) --》 得到 导致子进程异常终止的信号编号。


waitpid函数：	指定某一个进程进行回收。可以设置非阻塞。			waitpid(-1, &status, 0) == wait(&status);

	pid_t waitpid(pid_t pid, int *status, int options)

	参数：
		pid：指定回收某一个子进程pid

			> 0: 待回收的子进程pid

			-1：任意子进程

			0：同组的子进程。

		status：（传出） 回收进程的状态。

		options：WNOHANG 指定回收方式为，非阻塞。

	返回值：

		> 0 : 表成功回收的子进程 pid

		0 : 函数调用时， 参3 指定了WNOHANG， 并且，没有子进程结束。

		-1: 失败。errno


总结：

	wait、waitpid	一次调用，回收一个子进程。

	想回收多个。while 


进程间通信的常用方式，特征：

	管道：简单

	信号：开销小

	mmap映射：非血缘关系进程间

	socket（本地套接字）：稳定


管道： 

	实现原理： 内核借助环形队列机制，使用内核缓冲区实现。

	特质；	1. 伪文件

		2. 管道中的数据只能一次读取。

		3. 数据在管道中，只能单向流动。

	局限性：1. 自己写，不能自己读。

		2. 数据不可以反复读。

		3. 半双工通信。

		4. 血缘关系进程间可用。

![](../photos/src/linux/pipe.png)

pipe函数：	创建，并打开管道。

	int pipe(int fd[2]);

	参数：	fd[0]: 读端。

		fd[1]: 写端。

	返回值： 成功： 0

		 失败： -1 errno

管道的读写行为：

	读管道：
		1. 管道有数据，read返回实际读到的字节数。

		2. 管道无数据：	1）无写端，read返回0 （类似读到文件尾）

				2）有写端，read阻塞等待。

	写管道：
		1. 无读端， 异常终止。 （SIGPIPE导致的）

		2. 有读端：	1） 管道已满， 阻塞等待

				2） 管道未满， 返回写出的字节个数。


pipe管道： 用于有血缘关系的进程间通信。  ps aux | grep 		ls | wc -l	
 
	父子进程间通信：


	兄弟进程间通信：



fifo管道：可以用于无血缘关系的进程间通信。

	命名管道：  mkfifo 

	无血缘关系进程间通信：

		读端，open fifo O_RDONLY

		写端，open fifo O_WRONLY


文件实现进程间通信：

	打开的文件是内核中的一块缓冲区。多个无血缘关系的进程，可以同时访问该文件。


共享内存映射:

void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);		创建共享内存映射

	参数：
		addr： 	指定映射区的首地址。通常传NULL，表示让系统自动分配

		length：共享内存映射区的大小。（<= 文件的实际大小）

		prot：	共享内存映射区的读写属性。PROT_READ、PROT_WRITE、PROT_READ|PROT_WRITE

		flags：	标注共享内存的共享属性。MAP_SHARED、MAP_PRIVATE

		fd:	用于创建共享内存映射区的那个文件的 文件描述符。

		offset：默认0，表示映射文件全部。偏移位置。需是 4k 的整数倍。

	返回值：

		成功：映射区的首地址。

		失败：MAP_FAILED (void*(-1))， errno


int munmap(void *addr, size_t length);		释放映射区。

	addr：mmap 的返回值

	length：大小


使用注意事项：

	1. 用于创建映射区的文件大小为 0，实际指定非0大小创建映射区，出 “总线错误”。

	2. 用于创建映射区的文件大小为 0，实际制定0大小创建映射区， 出 “无效参数”。

	3. 用于创建映射区的文件读写属性为，只读。映射区属性为 读、写。 出 “无效参数”。

	4. 创建映射区，需要read权限。当访问权限指定为 “共享”MAP_SHARED是， mmap的读写权限，应该 <=文件的open权限。	只写不行。

	5. 文件描述符fd，在mmap创建映射区完成即可关闭。后续访问文件，用 地址访问。

	6. offset 必须是 4096的整数倍。（MMU 映射的最小单位 4k ）

	7. 对申请的映射区内存，不能越界访问。 

	8. munmap用于释放的 地址，必须是mmap申请返回的地址。

	9. 映射区访问权限为 “私有”MAP_PRIVATE, 对内存所做的所有修改，只在内存有效，不会反应到物理磁盘上。

	10.  映射区访问权限为 “私有”MAP_PRIVATE, 只需要open文件时，有读权限，用于创建映射区即可。


mmap函数的保险调用方式：

	1. fd = open（"文件名"， O_RDWR）;

	2. mmap(NULL, 有效文件大小， PROT_READ|PROT_WRITE, MAP_SHARED, fd, 0);


父子进程使用 mmap 进程间通信：

	父进程 先 创建映射区。 open（ O_RDWR） mmap( MAP_SHARED );

	指定 MAP_SHARED 权限

	fork() 创建子进程。

	一个进程读， 另外一个进程写。

无血缘关系进程间 mmap 通信：  				【会写】

	两个进程 打开同一个文件，创建映射区。

	指定flags 为 MAP_SHARED。

	一个进程写入，另外一个进程读出。

	【注意】：无血缘关系进程间通信。mmap：数据可以重复读取。

					fifo：数据只能一次读取。

匿名映射：只能用于 血缘关系进程间通信。

	p = (int *)mmap(NULL, 40, PROT_READ|PROT_WRITE, MAP_SHARED|MAP_ANONYMOUS, -1, 0);

信号共性：

	简单、不能携带大量信息、满足条件才发送。

信号的特质：

	信号是软件层面上的“中断”。一旦信号产生，无论程序执行到什么位置，必须立即停止运行，处理信号，处理结束，再继续执行后续指令。

	所有信号的产生及处理全部都是由【内核】完成的。

信号相关的概念：

	产生信号：

		1. 按键产生

		2. 系统调用产生

		3. 软件条件产生

		4. 硬件异常产生

		5. 命令产生

	概念：
		未决：产生与递达之间状态。  

		递达：产生并且送达到进程。直接被内核处理掉。

		信号处理方式： 执行默认处理动作、忽略、捕捉（自定义）


		阻塞信号集（信号屏蔽字）： 本质：位图。用来记录信号的屏蔽状态。一旦被屏蔽的信号，在解除屏蔽前，一直处于未决态。

		未决信号集：本质：位图。用来记录信号的处理状态。该信号集中的信号，表示，已经产生，但尚未被处理。

信号4要素：

	信号使用之前，应先确定其4要素，而后再用！！！

	编号、名称、对应事件、默认处理动作。

kill命令 和 kill函数：

	int kill（pid_t pid, int signum）

	参数：
		pid: 	> 0:发送信号给指定进程

			= 0：发送信号给跟调用kill函数的那个进程处于同一进程组的进程。

			< -1: 取绝对值，发送信号给该绝对值所对应的进程组的所有组员。

			= -1：发送信号给，有权限发送的所有进程。

		signum：待发送的信号

	返回值：
		成功： 0

		失败： -1 errno


alarm 函数：使用自然计时法。

	定时发送SIGALRM给当前进程。

	unsigned int alarm(unsigned int seconds);

		seconds：定时秒数

		返回值：上次定时剩余时间。

			无错误现象。

		alarm（0）； 取消闹钟。

	time 命令 ： 查看程序执行时间。   实际时间 = 用户时间 + 内核时间 + 等待时间。  --》 优化瓶颈 IO


setitimer函数：

	int setitimer(int which, const struct itimerval *new_value, struct itimerval *old_value);

	参数：
		which：	ITIMER_REAL： 采用自然计时。 ——> SIGALRM

			ITIMER_VIRTUAL: 采用用户空间计时  ---> SIGVTALRM

			ITIMER_PROF: 采用内核+用户空间计时 ---> SIGPROF
		
		new_value：定时秒数

		           类型：struct itimerval {

               				struct timeval {
               					time_t      tv_sec;         /* seconds */
               					suseconds_t tv_usec;        /* microseconds */

           				}it_interval;---> 周期定时秒数

               				 struct timeval {
               					time_t      tv_sec;         
               					suseconds_t tv_usec;        

           				}it_value;  ---> 第一次定时秒数  
           			 };

		old_value：传出参数，上次定时剩余时间。
	
		e.g.
			struct itimerval new_t;	
			struct itimerval old_t;	

			new_t.it_interval.tv_sec = 0;
			new_t.it_interval.tv_usec = 0;
			new_t.it_value.tv_sec = 1;
			new_t.it_value.tv_usec = 0;

			int ret = setitimer(&new_t, &old_t);  定时1秒

	返回值：
		成功： 0

		失败： -1 errno


其他几个发信号函数：

	int raise(int sig);

	void abort(void);


信号集操作函数：

	sigset_t set;  自定义信号集。

	sigemptyset(sigset_t *set);	清空信号集

	sigfillset(sigset_t *set);	全部置1

	sigaddset(sigset_t *set, int signum);	将一个信号添加到集合中

	sigdelset(sigset_t *set, int signum);	将一个信号从集合中移除

	sigismember（const sigset_t *set，int signum); 判断一个信号是否在集合中。 在--》1， 不在--》0

设置信号屏蔽字和解除屏蔽：

	int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);

		how:	SIG_BLOCK:	设置阻塞

			SIG_UNBLOCK:	取消阻塞

			SIG_SETMASK:	用自定义set替换mask。

		set：	自定义set

		oldset：旧有的 mask。

查看未决信号集：

	int sigpending(sigset_t *set);

		set： 传出的 未决信号集。

【信号捕捉】：

	signal();

	【sigaction();】 重点！！！

		

信号捕捉特性：

	1. 捕捉函数执行期间，信号屏蔽字 由 mask --> sa_mask , 捕捉函数执行结束。 恢复回mask

	2. 捕捉函数执行期间，本信号自动被屏蔽(sa_flgs = 0).

	3. 捕捉函数执行期间，被屏蔽信号多次发送，解除屏蔽后只处理一次！


借助信号完成 子进程回收。


	
守护进程：

	daemon进程。通常运行与操作系统后台，脱离控制终端。一般不与用户直接交互。周期性的等待某个事件发生或周期性执行某一动作。

	不受用户登录注销影响。通常采用以d结尾的命名方式。


守护进程创建步骤：

	1. fork子进程，让父进程终止。

	2. 子进程调用 setsid() 创建新会话

	3. 通常根据需要，改变工作目录位置 chdir()， 防止目录被卸载。

	4. 通常根据需要，重设umask文件权限掩码，影响新文件的创建权限。  022 -- 755	0345 --- 432   r---wx-w-   422

	5. 通常根据需要，关闭/重定向 文件描述符

	6. 守护进程 业务逻辑。while（）