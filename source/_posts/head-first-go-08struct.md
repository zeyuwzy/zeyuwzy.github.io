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

