---
title: Head First Go - 06切片
date: 2022-08-02 12:22:00
tags:
    - Go
---

## 切片
切片是Go的数据结构,底层基于数组实现，可以理解为数组的视图,和python切片使用差不多

切片不用指定数组大小，其他与声明数组变量语法相同

切片通过`make()`去创建切片的类型和长度

```go
package main

import "fmt"

func main() {
	var mySlice []int;	//声明切片
	mySlice = make([]int, 3) //创建三个整形的切片

	testSlice := make([]string,2) //短变量声明

	mySlice[0] = 0;
	testSlice[0] = "aaa";

	fmt.Println(mySlice[0])
	fmt.Println(testSlice[0])
}

```

## 切片字面量
和数组字面量差不多,而且不用调用`make`,也不用声明长度
```go
	testSlice := []string{"aaa", "bbb"}
```
## 切片运算符
和python一样(Go索引不能有负数,不能像Python一样反着切)underlyingArray[i:j]作为切片的运算符，生成的切片实际上包含元素underlyingArray[i]到元素underlyingArray[j-1]

- 切片运算符默认需要两个索引。如果你忽略了开始的索引，0(数组的第一个元素)会被使用
- 如果你忽略了结束的索引，从底层数组的开始索引到数组结尾之间的所有元素都会被包含到结果切片中
- 因为切片是底层数组的视图,修改底层数组的值也会影响切片,修改切片的值也会因影响底层数组,所以一般使用make或切片字面量创建切片，这样就不用关心底层数组

```go
package main

import "fmt"

func main() {

	testArr := [4]string{
			"aaa",
			"bbb",
			"ccc",
			"ddd",
		}

	testSlice := testArr[0:2]

	for _,value := range testSlice {
		fmt.Println(value)
	}
}
```
## append
append可以给切片添加元素，

切片的底层数组并不能增长大小。如果数组没有足够的空间来保存新的元素，所有的元素会被拷贝至一个新的更大的数组，并且切片会被更新为引用这个新的数组。但是由于这些场景都发生在append函数内部，无法知道返回的切片与传入append函数的切片是否具有相同的底层数组。如果你保留了两个切片，会导致一些非预期的错误,所以我们调用append函数，惯例是将函数的返回值赋给你传入的那个切片变量

```go
package main

import "fmt"

func main() {

	s1 := []string{"s1", "s1"}		
	s2 := append(s1, "s2", "s2")
	s3 := append(s2, "s3", "s3")
	s4 := append(s3, "s4", "s4")

	s4[0] = "wtf"

	fmt.Println(s1)
	fmt.Println(s2)
	fmt.Println(s3)
	fmt.Println(s4) //s3 s4切片共用一个底层数组
	
	//正常使用
	s5 := []string{"s5", "s5"}
	s5 := append(s5, "s6", "s6")
}
```
## 切片零值
不像数组，切片变量自己也有0值:nil。一个没有赋值的切片变量 值为nil

```go
package main

import "fmt"

func main() {
	var s1[]string //声明一个没有赋值的切片变量,初始值为nil
	s2 := []string{} //这个切片已经赋值了,底层是长度为0的数组
	s3 := make([]string, 0) //这个切片已经赋值了,底层是长度为0的数组
	
	fmt.Printf("%#v\n", s1)//nil
	fmt.Printf("%#v\n", s2) 
	fmt.Printf("%#v\n", s3) 

	fmt.Println(len(s1))
	fmt.Println(len(s2))
	fmt.Println(len(s3))
}
```

## 可变长参数
可变长参数函数的最后一个参数接收一个切片类型的变长参数，这个切片可以被函数当作普通切片来处理

如果我们不提供变长参数，它不会返回错误，函数会收到一个空的切片

仅仅函数定义中的最后一个参数可以是可变长参数,不能把它放到必需参数之前

让含有可变长参数的函数接受切片,可以在传入的切片变量后增加省略号(...)

```go
package main

import (
	"fmt"
	"reflect"
)

func test(val ...int) {
	fmt.Println(reflect.TypeOf(val))
	fmt.Println(val)
}

func main() {
	test(1, 2, 3)
	slice := []int{1, 2, 3}
	test(slice...) //可变参数直接接收一个切片
}
```