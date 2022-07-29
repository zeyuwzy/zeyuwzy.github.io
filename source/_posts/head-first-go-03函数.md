---
title: Head First Go - 03函数
date: 2022-07-25 08:39:00
tags:
    - Go
---

## 函数
### 函数定义和调用
> func 函数名(参数名 类型) 返回类型 {  
>	return  
>}

Go不像C++有缺省参数,如果希望调用函数来接受参数，必须在函数声明中声明一个或多个参数，包括每个参数的类型。所传递的参数的数量和类型必须始终与规定的参数的数量和类型相匹配，否则将出现编译错误

Go要求带返回类型的函数结尾必须有return

目前没有发现Go中的函数必须要声明,函数定义的顺序不影响函数使用

```go
package main

import (
	"fmt"
)

func main() {
	noReturn()
	rc := haveReturn()
	fmt.Println("rc=", rc)
}

func noReturn() {
	fmt.Printf("no return\n")
}

func haveReturn() int {
	fmt.Printf("have return\n")
	return 0
}


```

### 多个返回值

```go
package main

import (
	"fmt"
	"log"
)

func abc(num int) (rc int, err error) { //返回值可以写变量名称方便阅读代码，多个返回值必须加括号
	if num > 5 {
		return 0, nil
	}
	return -1, fmt.Errorf("bad num")
}

func main() {

	rc, err := abc(7)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("rc=%d\n", rc)
}
```
## 指针
和C差不多，只不过C声明变量时类型在变量名称前面(int* rc),Go类型在变量名称后面(var rc *int), *的位置也相反

### Go是一种“值传递”语言,函数形参从函数调用中接收实参的副本

### 在Go中，返回一个指向函数局 部变量的指针是可以的。即使该变量不在作用域内，只要你仍然拥有指针，Go将确保你仍然可以访问该值

```go
package main

import (
	"fmt"
)

func retPointer() *int {
	val := 5
	return &val
}

func main() {
	var rc *int = retPointer()
	fmt.Println(*rc)
}
```

## Printf Sprintf
和C差不多

```go 
package main

import (
	"fmt"
)

func main() {

	fmt.Printf("A float: %f\n", 3.1415)			//浮点
	fmt.Printf("An integer: %d\n", 15)			//整数
	fmt.Printf("A string: %s\n", "hello")			//字符串
	fmt.Printf("A boolean: %t\n", false)			//bool
	fmt.Printf("Values: %v %v %v\n", 1.2, "\t", true)	//任意类型
	fmt.Printf("Values: %#v %#v %#v\n", 1.2, "\t", true)	//任意类型，按代码中显示的格式进行格式化
	fmt.Printf("Types: %T %T %T\n", 1.2, "\t", true)	//返回值的类型
	fmt.Printf("Percent sign: %%\n")			//返回%
}

```