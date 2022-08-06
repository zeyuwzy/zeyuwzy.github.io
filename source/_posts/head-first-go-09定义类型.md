---
title: Head First Go - 09定义类型
date: 2022-08-06 15:50:00
tags:
    - Go
---

## 定义基础类型
```go
type myTypeA float64
type myTypeB float64
```

虽然myTypeA,和myTypeB都是同一基础类型，但是不能把另一个类型的值赋给它

```go
package main

import "fmt"

type myTypeA float64
type myTypeB float64

func main() {
	var a myTypeA
	var b myTypeB

	a = 1.1 //基础类型可以直接赋值，不过最好也用转换
	b = 2.1

	fmt.Println(a)
	fmt.Println(b)

    //不能直接赋值,需要类型转换
	//a = b
	a = myTypeA(b)
}
```

- 一个定义类型提供所有与基础类型相同的运算
- 一个定义类型可以被用来与字面值一起用于运算
- 但是定义类型不能用来与不同类型的值一起运算，即使它们是来自相同的基础类型

## 定义方法
可以为指定的类型定义方法

一旦方法被定义在了某个类型，它就能被该类型的任何值调用

`func` (`接收器` `类型`) `函数名`() { }

```go
package main

import "fmt"

type myTypeA float64

//将hello定义在了myTypeA上
func (m myTypeA) hello() {
	fmt.Println("hello", m)
}

func main() {
	var test myTypeA = 1.1
	test.hello()
}
```

- 接收器的参数类似C++的this指针
- 按照惯例，Go开发者通常使用一个字母作为名称——小写的接收器类型名称的首字母
- 方法和类型必须定义在同一包中
- Go使用接收器参数来代替self和this，self和this是隐含的，而Go显式地声明一个接收器参数

## 导出方法
就像其他的函数，方法名称以大写字母开头，则认为是导出的

## 指针类型的接收器参数
就像其他的参数一样，接收器参数接收一个原值的拷贝。如果你的方法需要修改接收器，你应该在接收器参数上使用指针类型，并且修改指针指向的值

```go
package main

import "fmt"

type myTypeA float64

func (m *myTypeA) helloA() {
	*m = 0
	fmt.Println("hello", *m)
}

func main() {
	var test myTypeA = 1.1
    /*
    不需要修改方法的调用。当你用一个非指针的变量调用一个需要指针的接收器的方法的时候，Go会自动为你将非指针类型转换为指针类型。同样指针类型也会自动转换为非指针类型，如果你调用一个要求值类型的接收器，Go会自动帮你获取指针指向的值，然后传递给方法
    */
	test.helloA()
}
```

## 其他
你需要将值保存在变量中，允许Go能得到一个指向它的指针
```go
package main

import "fmt"

type myTypeA float64


func (m *myTypeA) hello() {
	*m = 0
	fmt.Println("hello", *m)
}

func (m myTypeA) hi() {
	fmt.Println("hi", m)
}

func main() {
	var test myTypeA = 15
	fmt.Println(&test)
	test.hi()
	test.hello()

	myTypeA(15).hi()
	//myTypeA(15).hello() cannot take the address of myTypeA(15) 
	//fmt.Println(&myTypeA(15)) cannot take the address of myTypeA(15) 
}

```