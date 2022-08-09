---
title: Head First Go - 11接口
date: 2022-08-08 09:30:00
tags:
    - Go
---

## 接口
一个接口是特定值预期具有的一组方法

如果一个类型包含接口中声明的所有方法，那么它可以在任何需要接口的地方使用，而不需要更多的声明

方法名称、参数类型(可能没有)和返回值(可能没有)都需要与接口中定义的一致。除了接口中列出的方法之外，类型还可以有更多的方法，但是它不能缺少接口中的任何方法，否则就不满足那个接口

一个类型可以满足多个接口，一个接口(通常应该)可以有多个类型满足它

    type 接口名 interface {  
        funcA  
        funcB  
    }

```go
package main

import (
	"fmt"
)
//定义一个接口，有两个方法
type myInterface interface {
	outPut()
	outPutWithString(str string)
}

type myType int

func (m myType) outPut() {
	fmt.Println(m)
}

func (m myType) outPutWithString(str string) {
	fmt.Println(m, str)
}

type myAnotherType string

func (m myAnotherType) outPut() {
	fmt.Println(m)
}


func (m myAnotherType) outPutWithString(str string) {
	fmt.Println(m, str)
}

func main() {
	var intf myInterface

    //这两个类型恰好都满足这个接口，就可以赋值成功

	intf = myType(15)
	intf.outPut()

	intf = myAnotherType("aaa")
	intf.outPut() 
}

```
## 接口变量
接口变量和普通变量没什么区别，也可以当函数参数或返回值

如果一个类型声明了指针接收器方法，你就只能将那个类型的指针传递给接口变量

## 类型断言
可以通过类型断言将接口变量转换为它对应的类型变量

`类型变量, bool = 接口变量.(类型)`

bool等于true说明断言成功

```go
package main

import (
	"fmt"
)

type myInterface interface {
	outPut()
}

type myType int

func (m myType) outPut() {
	fmt.Println(m)
}

type myAnotherType string

func (m myAnotherType) outPut() {
	fmt.Println(m)
}


func (m myAnotherType) outPutWithString(str string) {
	fmt.Println(m, str)
}

func testFunc(intf myInterface) {
	intf.outPut()
	another,ok := intf.(myAnotherType)
	if ok {
		another.outPutWithString("ok")
	} else {
		fmt.Println("type err")
	}
}

func main() {
	testFunc(myType(15))
	testFunc(myAnotherType("hhh"))
}
```

## error接口

error是一个接口类型 

    type error interface {  
           Error() string  
    }  

error类型像int或者string一样是一个“预定义标识符”，它不属于任何包。它是“全局块”的一部分，这意味着它在任何地方可用，不用考虑当前包信息

可以自定义满足error接口的类型

```go
package main

import (
	"fmt"
)

type myErrorType string

func (m myErrorType) Error() string {
	return string(m)
}

func main() {
	var err error //声明一个error接口变量

	err = myErrorType("wtf") //我们的类型方法满足error接口,可以赋值

	fmt.Println(err.Error())
}
```

## stringer

fmt包定义了fmt.Stringer接口:允许任何类型决定在输出时如何展示

    type Stringer interface {  
        String() string  
    }  

许多在fmt包中的函数都会判断传入的参数是否满足stringer接口，如果满足就调用String方法。这些函数包括Print、Println和Printf等

```go
package main

import (
	"fmt"
)

type myType string

func (m myType) String() string {
	return "--- " + string(m) + " ---"
}

func main() {
	value := myType("aaa")
	fmt.Println(value)
}
```

## 空接口
fmt.Println能接收任意类型,因为它声明了空接口`Interface{}`

Interface{}类型称为空接口，用来接收任何类型的值。不需要实现 任何方法来满足空接口，所以所有的类型都满足它

```go
package main

import (
	"fmt"
)

type myType string

func (m myType) myTpyeFunc() {
	fmt.Println("myTypeFunc", m)
}

//空接口作为函数参数
func test(empty interface{}) {
	fmt.Println(empty)

	mt, ok := empty.(myType) //空接口无法调用任何方法,通过断言转换成对应的类型
	if ok {
		mt.myTpyeFunc()
	}
}

func main() {
	test(123)
	test(myType("abc"))
}

```