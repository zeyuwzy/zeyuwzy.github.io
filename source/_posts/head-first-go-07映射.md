---
title: Head First Go - 07映射
date: 2022-08-05 11:00:00
tags:
    - Go
---

## 映射
映射可以使用任意类型的键(只要这个类型可以使用==来比较)

`var` testMap `map`[`keyType`]`valueType`

与切片一样，声明一个映射变量并不会自动创建一个映射，你需要调用make函数

```go
package main

import "fmt"

func main() {
	var testMap map[int]string //声明一个映射变量
	testMap = make(map[int]string) //创建一个映射变量

	testMap[0] = "aaa"
	fmt.Println(testMap[0])
}
```

## 映射字面量
`map`[`keyType`]`valueType`{ k1:v1, k2:v2 }

```go
package main

import "fmt"

func main() {
	emptyMap := map[int]string{} //空映射
	testMap := map[int]string{1:"aaa", 2:"bbb"} 
	fmt.Println(emptyMap)
	fmt.Println(testMap)
}
```

## 映射零值
就像跟切片一样，映射变量的零值是nil。如果你声明了一个映射变量但是未赋值，它的值是nil。那意味着没有映射存在来增加键或者值。如果你尝试这么做，会产生一个panic。但是nil的切片可以通过append内置函数被赋值

如果你访问一个没有赋值过的key，你会得到一个value对应的零值

```go
package main

import "fmt"

func main() {
	var nilMap map[int]string
	emptyMap := map[int]string{}

	fmt.Printf("%#v\n", emptyMap)
	fmt.Printf("%#v\n", nilMap)
	fmt.Printf("%#v\n", emptyMap[1])
}
```

## 区分赋值为0和零值
访问映射键的时候可选地获取第2个布尔类型的值。如果这个键已经被赋过值，那么返回true，否则返回false

```go
package main

import "fmt"

func main() {
	myMap := map[int]string{1:"aaa", 2:"bbb"}

	val1, oK := myMap[1]
	fmt.Println(val1, oK)

	val3, oK := myMap[3]
	fmt.Println(val3, oK)
}
```

## delete
通过内置函数delete删除映射的一个value

```go
package main

import "fmt"

func main() {
	testMap := map[string]string{"test" :"testVal"}

	val, ok := testMap["test"]
	fmt.Println(ok,val)

	delete(testMap, "test")

	_, ok = testMap["test"]
	fmt.Println(ok)
}
```

## 遍历映射
可以使用`for key,value := range`遍历映射,映射的遍历是无序的

