---
title: Head First Go - 13goroutine
date: 2022-08-12 08:40:00
tags:
    - Go
---

## goroutine
在Go中，并发任务称为goroutine。其他编程语言有一个类似的概念，叫作线程，但是goroutine比线程需要更少的计算机内存

`go 函数()`

每个Go程序的main函数都是使 用goroutine启动的

goroutine无法接收函数的返回值

```go
package main

import (
	"fmt"
	"time"
)

func taskA() {
	for i := 0; i < 50; i++ {
		fmt.Println(`A`,i)
		time.Sleep(time.Microsecond)
	}
}

func taskB() {
	for i := 0; i < 50; i++ {
		fmt.Println(`B`,i)
		time.Sleep(time.Microsecond)
	}
}

func main() {
	go taskA()
	go taskB()
	time.Sleep(time.Second * 5)
}

```
## channel
channel可以在不同goroutine传递消息

每个channel只携带特定类型的值

创建channel需要调用内置的make函数

```go
var myChannel chan int
myChannel = make(chan int)
```

## 使用channel发送和接收值
- 发送 `myChannel <- value`
- 接收 `value <- myChannel`

```go
package main

import (
	"fmt"
	"time"
)

func taskA(myCh chan int) {
	myCh <- 15
}

func main() {
	myCh := make(chan int)
	go taskA(myCh)
	fmt.Println(<-myCh)
}
```

默认情况下发送方会阻塞当前goroutine,直到发送的值被接收;接收方也会阻塞当前goroutine，直到channel被接收

所以跨goroutine要保证channel有发就有收,并且要按收发的顺序来,避免出现死锁

```go
package main

import (
	"fmt"
)

func taskA(myCh chan int) {
	myCh <- 1
	myCh <- 2
	myCh <- 3
}

func taskB(myCh chan int) {
	myCh <- 4
	myCh <- 5
	myCh <- 6
}

func main() {
	chA := make(chan int)
	chB := make(chan int)

	go taskA(chA)
	go taskB(chB)

	fmt.Println(<- chA)
	fmt.Println(<- chB)

	fmt.Println(<- chA)
	fmt.Println(<- chB)

	fmt.Println(<- chA)
	fmt.Println(<- chB)
    //1 2 3 4 5 6
}
```