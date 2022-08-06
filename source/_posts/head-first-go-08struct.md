---
title: Head First Go - 08struct
date: 2022-08-05 20:00:00
tags:
    - Go
---

## struct

```go
package main

import "fmt"

func main() {
	var myStruct struct {
		num int
		str string
	}

	fmt.Printf("%#v\n", myStruct)

	myStruct.num = 5
	fmt.Println(myStruct.num)
}
```

## 定义类型
`type` 类型名 `基础类型`

不要使用一个已经存在的类型名称作为变量的名称

```go
package main

import "fmt"

type myType struct {
	name string
}

func main() {
	var st myType
	st.name = "zeyu"

	fmt.Println(st.name)
}
```

## 指针访问struct字段
使用点运算符在struct指针和struct上都可访问字段

这里和C就有区别了，C取普通struct变量的成员用`.`,取指向struct的指针变量指向的成员要用`->`,Go可以都用`.`

如果函数需要修改struct或者struct过大，应该向函数传递，返回指针

```go
package main

import "fmt"

type myType struct {
	name string
}

func main() {

	var st myType
	st.name = "zeyu"

	var pointer *myType = &st

	fmt.Println((*pointer).name)
	fmt.Println(pointer.name)
}
```

## struct 导出
定义类型的名称首字母必须大写才能导出该类型

struct字段的名称首字母必须大写才能导出该字段
```go
type MyType struct {
	Name string//能导出
	age int//不能导出
}
```

## struct 字面量
和映射差不多
```go
package main

import "fmt"

type myType struct {
	name string
	age int
}

func main() {
	st := myType{name : "aaa", age: 15}
	fmt.Println(st.name)
}
```

## struct 嵌套
没法直接用字面量`{ {} }`给嵌套的结构体赋值 

```go
package main

import "fmt"

type stb struct {
	b int
}

type sta struct {
	a int
	sub stb 
}

func main() {
	st := sta{a: 1}

	st.sub = stb{b : 15} //直接填充整个子struct
	fmt.Println(st.sub.b)

	st.sub.b = 100 //给子strcut成员赋值
	fmt.Println(st.sub.b)
}
```
## 匿名struct
一个内部struct使用匿名字段的方式存储在了外部的struct中，这被称为嵌入了外部struct。嵌入struct的字段被提升到了外部struct，可以像访问外部struct的字段一样访问它们

```go
package main

import "fmt"

type stb struct {
	b int
}

type sta struct {
	a int
	stb //只写类型名 
}

func main() {
	st := sta{a: 1}
    //当你声明一个匿名字段时，你可以使用字段类型名称作为字段名称
	st.stb = stb{b : 15}
	fmt.Println(st.stb.b)

	st.stb.b = 100
	fmt.Println(st.stb.b)

    //可以像访问外部字段一样访问嵌入的strcut字段
	st.b = 50
	fmt.Println(st.b)
}
```