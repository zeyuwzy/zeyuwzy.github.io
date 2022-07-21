---
title: Head First Go - 语法基础
date: 2022-07-19 23:19:11
tags:
    - Go
---

开了一个新坑 **Ready ~~~ Go!** ( ^ @ ^ )
***

## 认识Go
一般Go文件都是由以下三部分组成
1. package子句
2. import语句
3. 代码片段


```go
package main //表示文件所有的代码都属于main包

import "fmt" //包含了fmt包

func main() { //首先运行main函数
	fmt.Println("Hello, Go")
}
```

[代码测试网站](https://go.dev/play/)

## 调用函数
包名.函数名(参数)
## 字符串
和C区别不大诶
## 符文(rune)
就是单个字符，Go中用Unicode存储符文
## 布尔
true false
## 数字
字面意思
## 数学运算比较 
和C差不多，比较结果返回bool
## 类型
Go也是静态类型，在运行之前类型就要确定,如果使用了错误类型Go会编译失败

`reflect`包中的`TypeOf`可以输出类型

```go
package main

import (
	"fmt"
	"reflect"
)

func main() {
	fmt.Println(reflect.TypeOf(123))
	fmt.Println(reflect.TypeOf(123.456))
	fmt.Println(reflect.TypeOf("zeyu"))
	fmt.Println(reflect.TypeOf('z'))
	fmt.Println(reflect.TypeOf(false))

}
```

## 声明变量
`var` 变量名称 类型
```go
package main

import (
	"fmt"
)

func main() {
	var test int //单个变量声明
	test = 4     //单个变量赋值

	var testa, testb string     //多个变量声明
	testa, testb = "hhh", "xxx" //多个变量赋值

	var testc, testd float64 = 1.1, 1.2 //声明同时赋值
	var teste, testf = 5, "www"         //声明同时赋值 省略类型

	fmt.Println(test)
	fmt.Println(testa)
	fmt.Println(testb)
	fmt.Println(testc)
	fmt.Println(testd)
	fmt.Println(teste)
	fmt.Println(testf)
}
```

## 零值
对于没有赋值的变量默认是零值
字符串变量的零值是空字符串，bool变量的零值是false,int,float64的零值是0

## 短变量声明
如果声明的时候就知道变量的初值，可以使用短变量声明，省略`val`和类型 赋值符号变为`:=`

```go
package main

import (
	"fmt"
)

func main() {
	test := 5 //短变量声明
	fmt.Println(test)
}
```

## 变量声明规范
1. 一个变量只能声明一次
2. 不能给未声明的变量赋值
3. 只能给变量赋相同类型的值
4. 声明同时赋值要求变量数量和值一一对应
5. 所有声明的变量都必须在程序中使用,否则因该删除声明

## 命名规则
名称必须以字母开头，可以有任意数量的额外字母和数字  
变量、函数、类型的名称如果以大写字母开头，则认为它是导出的,可以通过其他包访问，否则认为是未导出的，只能在当前包使用  
名称一般用驼峰式  

## 转换
使用不同类型的值直接比较会报错  
赋值的类型和变量类型不匹配也会报错  

解决办法是使用转换  `类型(变量)`

## 运行Go
```bash
go fmt hello.go //重新格式化 
go build hello.go //编译为二进制
./hello
```

go run hello.go //编译并运行，不保存可执行文件