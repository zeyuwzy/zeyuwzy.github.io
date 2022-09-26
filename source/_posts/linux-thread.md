---
title: Linux Thread
date: 2021-05-18 10:00:00
tags:
    - Linux
---

线程概念：

	进程：有独立的 进程地址空间。有独立的pcb。	分配资源的最小单位。

	线程：有独立的pcb。没有独立的进程地址空间。	最小单位的执行。

	ps -Lf 进程id 	---> 线程号。LWP  --》cpu 执行的最小单位。

线程共享：

	独享 栈空间（内核栈、用户栈）

	共享 ./text./data ./rodataa ./bsss heap  ---> 共享【全局变量】（errno）

线程控制原语：

	pthread_t pthread_self(void);	获取线程id。 线程id是在进程地址空间内部，用来标识线程身份的id号。

		返回值：本线程id


	检查出错返回：  线程中。

		fprintf(stderr, "xxx error: %s\n", strerror(ret));


	int pthread_create(pthread_t *tid, const pthread_attr_t *attr, void *(*start_rountn)(void *), void *arg); 创建子线程。

		参1：传出参数，表新创建的子线程 id

		参2：线程属性。传NULL表使用默认属性。

		参3：子线程回调函数。创建成功，ptherad_create函数返回时，该函数会被自动调用。
		
		参4：参3的参数。没有的话，传NULL

		返回值：成功：0

			失败：errno


	循环创建N个子线程：

		for （i = 0； i < 5; i++）

			pthread_create(&tid, NULL, tfn, (void *)i);   // 将 int 类型 i， 强转成 void *， 传参。	


	void pthread_exit(void *retval);  退出当前线程。

		retval：退出值。 无退出值时，NULL

		exit();	退出当前进程。

		return: 返回到调用者那里去。

		pthread_exit(): 退出当前线程。


	int pthread_join(pthread_t thread, void **retval);	阻塞 回收线程。

		thread: 待回收的线程id

		retval：传出参数。 回收的那个线程的退出值。

			线程异常借助，值为 -1。

		返回值：成功：0

			失败：errno

	int pthread_detach(pthread_t thread);		设置线程分离

		thread: 待分离的线程id

	
		返回值：成功：0

			失败：errno	

	int pthread_cancel(pthread_t thread);		杀死一个线程。  需要到达取消点（保存点）

		thread: 待杀死的线程id
		
		返回值：成功：0

			失败：errno

		如果，子线程没有到达取消点， 那么 pthread_cancel 无效。

		我们可以在程序中，手动添加一个取消点。使用 pthread_testcancel();

		成功被 pthread_cancel() 杀死的线程，返回 -1.使用pthead_join 回收。


	线程控制原语					进程控制原语


	pthread_create()				fork();

	pthread_self()					getpid();

	pthread_exit()					exit(); 		/ return 

	pthread_join()					wait()/waitpid()

	pthread_cancel()				kill()

	pthread_detach()
	

线程属性：

	设置分离属性。

	pthread_attr_t attr  	创建一个线程属性结构体变量

	pthread_attr_init(&attr);	初始化线程属性

	pthread_attr_setdetachstate(&attr,  PTHREAD_CREATE_DETACHED);		设置线程属性为 分离态

	pthread_create(&tid, &attr, tfn, NULL); 借助修改后的 设置线程属性 创建为分离态的新线程

	pthread_attr_destroy(&attr);	销毁线程属性




线程同步：

	协同步调，对公共区域数据按序访问。防止数据混乱，产生与时间有关的错误。

锁的使用：

	建议锁！对公共数据进行保护。所有线程【应该】在访问公共数据前先拿锁再访问。但，锁本身不具备强制性。


使用mutex(互斥量、互斥锁)一般步骤：

	pthread_mutex_t 类型。 

	1. pthread_mutex_t lock;  创建锁

	2  pthread_mutex_init; 初始化		1

	3. pthread_mutex_lock;加锁		1--	--> 0

	4. 访问共享数据（stdout)		

	5. pthrad_mutext_unlock();解锁		0++	--> 1

	6. pthead_mutex_destroy；销毁锁


	初始化互斥量：

		pthread_mutex_t mutex;

		1. pthread_mutex_init(&mutex, NULL);   			动态初始化。

		2. pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;	静态初始化。

	注意事项：

		尽量保证锁的粒度， 越小越好。（访问共享数据前，加锁。访问结束【立即】解锁。）

		互斥锁，本质是结构体。 我们可以看成整数。 初值为 1。（pthread_mutex_init() 函数调用成功。）

		加锁： --操作， 阻塞线程。

		解锁： ++操作， 换醒阻塞在锁上的线程。

		try锁：尝试加锁，成功--。失败，返回。同时设置错误号 EBUSY

restrict关键字：

	用来限定指针变量。被该关键字限定的指针变量所指向的内存操作，必须由本指针完成。

【死锁】：

	是使用锁不恰当导致的现象：

		1. 对一个锁反复lock。

		2. 两个线程，各自持有一把锁，请求另一把。


读写锁：

	锁只有一把。以读方式给数据加锁——读锁。以写方式给数据加锁——写锁。

	读共享，写独占。

	写锁优先级高。

	相较于互斥量而言，当读线程多的时候，提高访问效率

	pthread_rwlock_t  rwlock;

	pthread_rwlock_init(&rwlock, NULL);

	pthread_rwlock_rdlock(&rwlock);		try

	pthread_rwlock_wrlock(&rwlock);		try

	pthread_rwlock_unlock(&rwlock);

	pthread_rwlock_destroy(&rwlock);

条件变量：

	本身不是锁！  但是通常结合锁来使用。 mutex

	pthread_cond_t cond;

	初始化条件变量：

		1. pthread_cond_init(&cond, NULL);   			动态初始化。

		2. pthread_cond_t cond = PTHREAD_COND_INITIALIZER;	静态初始化。

	阻塞等待条件：

		pthread_cond_wait(&cond, &mutex);

		作用：	1） 阻塞等待条件变量满足

			2） 解锁已经加锁成功的信号量 （相当于 pthread_mutex_unlock(&mutex)）

			3)  当条件满足，函数返回时，重新加锁信号量 （相当于， pthread_mutex_lock(&mutex);）


	pthread_cond_signal(): 唤醒阻塞在条件变量上的 (至少)一个线程。

	pthread_cond_broadcast()： 唤醒阻塞在条件变量上的 所有线程。


	【要求，能够借助条件变量，完成生成者消费者】

信号量： 

	应用于线程、进程间同步。

	相当于 初始化值为 N 的互斥量。  N值，表示可以同时访问共享数据区的线程数。

	函数：
		sem_t sem;	定义类型。

		int sem_init(sem_t *sem, int pshared, unsigned int value);

		参数：
			sem： 信号量 

			pshared：	0： 用于线程间同步
					
					1： 用于进程间同步

			value：N值。（指定同时访问的线程数）


		sem_destroy();

		sem_wait();		一次调用，做一次-- 操作， 当信号量的值为 0 时，再次 -- 就会阻塞。 （对比 pthread_mutex_lock）

		sem_post();		一次调用，做一次++ 操作. 当信号量的值为 N 时, 再次 ++ 就会阻塞。（对比 pthread_mutex_unlock）




































	

		



























