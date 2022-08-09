---
title: Head First Go - 10封装
date: 2022-08-07 10:20:00
tags:
    - Go
---

## 封装
Go通过限制struct中成员是否能导出配合该strcut类型定义相关的方法实现封装

### setter
setter方法是用来设置封装字段值的方法

setter方法需要指针接收器

setter方法通常被命名为setX, X为被设置字段的名称

### getter
getter方法是用于获取封装字段的方法

getter方法通常被命名为X, X为被设置字段的名称 (一般习惯前面不加get)

```go
package abc

import (
	"errors"
	"unicode/utf8"
)

type Abc struct {
	name string
}

func (a *Abc) SetName(name string) error {
	if utf8.RuneCountInString(name) > 20 {
		return errors.New("out of len")
	}

	a.name = name
	return nil
}

func (a *Abc) Name() string {
	return a.name
}

```

## 嵌入 struct
当子struct嵌入到父strcut,子struct未导出的字段不会被提升

当子struct嵌入到父strcut,子struct导出的方法会被提升


```go
package main

import (
	"fmt"
	"abc"
)

type myType struct {
	abc.Abc
}

func main() {
	var st myType

	//Abc未导出的字段无法提升
	//fmt.Println(st.name) 

	//Abc导出的方法可以提升
	st.SetName("zeyu")
	fmt.Println(st.Name())
	//通过类型名访问也可以
	fmt.Println(st.Abc.Name())
}
```

注意当struct嵌入了多个子struct时，导出方法可能会有重名的情况,这时直接调用导出方法会无法编译通过,需要加上类型

```go
//abc包中的Abc和def包中的Def都定义了setName()和Name()
package main

import (
	"fmt"
	"abc"
	"def"
	"log"
)

type myType struct {
	abc.Abc
	def.Def
}

func main() {
	var st myType

	//Abc未导出的字段无法提升
	//fmt.Println(st.name) 

	//Abc导出的方法可以提升

	//重名
	err := st.SetName("zeyu")
	if err != nil {
		log.Fatal(err)
	}

	//重名
	fmt.Println(st.Name())

    /*
    正确使用
    err := st.Abc.SetName("zeyu")
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(st.Abc.Name())
    */

}
//./test.go:22:11: ambiguous selector st.SetName
//./test.go:27:16: ambiguous selector st.Name

```