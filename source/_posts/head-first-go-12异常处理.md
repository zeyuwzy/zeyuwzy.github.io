---
title: Head First Go - 12异常处理
date: 2022-08-10 08:40:00
tags:
    - Go
---
## 写在最前面
Go语言本身的设计不鼓励使用panic和recover,Go开发人员鼓励使用if和return语句,以及error值进行错误处理

## 延迟调用
声明defer的语句在函数所有调用执行之后,return之前执行

最先声明的defer最后执行

```go
package main

import "fmt"

func test() int {
	fmt.Println("abc")
	defer fmt.Println("000")
	defer fmt.Println("111")
	fmt.Println("def")
	defer fmt.Println("222")
	defer fmt.Println("333")
	return 0
}

func main() {
	test()
}
```
## 延迟调用在panic前完成
当程序出现panic时，当前函数停止运行，程序打印日志消息并崩溃。可以通过简单地调用内置的panic函数来引发panic

panic函数需要一个满足空接口的参数(也就是说，它可以是任何类型)。该参数将被转换为字符串(如果需要)，并作为panic日志信息的一部分打印出来

当程序出现panic时，所有延迟的函数调用仍然会被执行。如果有多个延迟调用，它们的执行顺序将与被延迟的顺序相反

```go
package main

import (
	"fmt"
)

func testOne() {
	defer fmt.Println("defer one")
	testTwo()
}

func testTwo() {
	defer fmt.Println("defer two")
	panic("die")
}

func main() {
	testOne()	
}
```

## 何时产生panic
调用panic应该留给“不可能的”情况,错误表示的是程序中的错误，而不是用户方面的错误

## recover函数
Go提供了一个内置的recover函数，可以阻止程序陷入panic

在正常程序执行过程中调用recover时，它只返回nil，而不执行其他操作

需要在程序处于panic状态时调用recover才能生效，recover就要配合defer一起使用

recover必须在defer函数中使用，但是不能被defer直接调用 `defer recover()`是不生效的

recover会恢复让程序继续执行,但是panic的那个函数从panic后都不会执行

```go
package main

import (
	"fmt"
)

func catchPanic() {
	recover()
}

func testOne() {
	defer catchPanic()
	panic("die")
    fmt.Println("not go there")
}

func main() {
	testOne()	
	fmt.Println("go on")
}
```
recover的返回值的类型也是interface{},要对panic值调用方法或执行其他操作，需要使用类型断言将其转换回其底层类型

```go
package main

import (
	"fmt"
)

func catchPanic() {
	p := recover() //p是空接口
	if p == nil { //没有panci时返回nil
		return
	}

	err, ok := p.(error)
	if (ok) {
		fmt.Println("catch", err.Error())
	} else {
		panic(p) //未知panic类型,恢复panic
	}
}


func main() {
	defer catchPanic()
	err := fmt.Errorf("wtf")
	panic(err) //传入一个error接口变量
}

```