---
title: Head First Go - 05数组
date: 2022-07-31 15:36:00
tags:
    - Go
---

## 数组定义

创建数组时，它所包含的所有值都初始化为数组所保存类型的零值

```go
package main

import "fmt"

func main() {
	var arr [3]string
	arr[0] = "aaa"
	arr[1] = "bbb"
	fmt.Println(arr[0])
	fmt.Println(arr[1])
	fmt.Println(arr[2])

}
```
## 数组字面量
可以用数组字面量初始化数组

`数组变量=[n]类型{ , , , }`

```go
package main

import "fmt"

func main() {
	var arr [3]string = [3]string{ //字面量可以换行，但是要在每一项前使用 ,
        "aaa",
		"bbb",
		"ccc",
        }

	test := [3]int{1, 2}

	fmt.Println(arr[0])
	fmt.Println(arr[1])
	fmt.Println(arr[2])

	fmt.Println(test[1])
	fmt.Println(test[2])

    fmt.Println(arr) // Println可以直接打印数组
    fmt.Printf("%#v", arr) //打印数组字面量

}

```

## 访问数组元素

数组下标越界会导致`panic`

- 可以使用`len()`获取数组长度
```go
	for i := 0; i < len(arr); i++ {
		fmt.Println(arr[i])
	}
```

- 使用`for ... range`关键字

```go
	for index, value := range arr { //index返回下标，value下标对应值
		fmt.Println(index, value)
	}
```

## 函数返回数组

这里和C不太一样，Go可以直接返回一个数组，而不是指针

```go
package main

import "fmt"

func returnArr() ([3]int, error) {
	return [3]int{1, 2, 3}, fmt.Errorf("wtf")
}

func main() {
	arr, _ := returnArr()

	for _, value := range arr {
		fmt.Println(value)
	}
}
```

## 其他

读文件
```go
package main

import (
	"bufio"
	"fmt"
	"log"
	"os"
)

func main() {
	file, err := os.Open(os.Args[1]) //从argv读取文件路径
	if err != nil {
		log.Fatal(err)
	}

	scanner := bufio.NewScanner(file)
	for scanner.Scan() { //从文件逐行读取，一直循环到文件末尾，返回false
		fmt.Println(scanner.Text())
	}

	err = file.Close()
	if err != nil {     //如果关闭文件失败
		log.Fatal(err)
	}

	if scanner.Err() != nil { //如果扫描文件失败
		log.Fatal(scanner.Err())
	}
}
```