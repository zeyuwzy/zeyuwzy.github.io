---
title: Head First Go - 15web应用程序
date: 2022-08-14 22:00:00
tags:
    - Go
---

## byte类型
是Go的基本类型之一,用于保存原始数据

```go
    //string与byte相互转换
	message := []byte("Hello")
    fmt.Println(message)
	fmt.Println(string([]byte{72, 101, 108, 108, 111}))
```

## web应用程序
```go
package main

import (
	"log"
	"net/http"
)

func write(writer http.ResponseWriter, message string) {
	_, err := writer.Write([]byte(message))
	if err != nil {
		log.Fatal(err)
	}
}

func englishHandler(writer http.ResponseWriter, request *http.Request) {
	write(writer, "Hello, web!")
}
func frenchHandler(writer http.ResponseWriter, request *http.Request) {
	write(writer, "Salut web!")
}
func hindiHandler(writer http.ResponseWriter, request *http.Request) {
	write(writer, "Namaste, web!")
}

func main() {
    /* 针对不同的路径请求调用不同的函数 */
	http.HandleFunc("/hello", englishHandler)
	http.HandleFunc("/salut", frenchHandler)
	http.HandleFunc("/namaste", hindiHandler)
    /* 在8080端口运行web服务 */
	err := http.ListenAndServe("localhost:8080", nil)
	log.Fatal(err)
}
```
## 并发请求Web页面，返回页面大小
```go
package main

import (
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
)

type Page struct {
	URL  string
	Size int
}

func responseSize(url string, channel chan Page) {
	fmt.Println("Getting", url)
	response, err := http.Get(url)
	if err != nil {
		log.Fatal(err)
	}
	defer response.Body.Close()
	body, err := ioutil.ReadAll(response.Body)
	if err != nil {
		log.Fatal(err)
	}
	channel <- Page{URL: url, Size: len(body)}
}

func main() {
	pages := make(chan Page)
	urls := []string{"https://example.com/",
		"https://golang.org/", "https://golang.org/doc"}
	for _, url := range urls {
		go responseSize(url, pages)
	}
	for i := 0; i < len(urls); i++ {
		page := <-pages
		fmt.Printf("%s: %d\n", page.URL, page.Size)
	}
}
```

## 函数变量
和C的函数指针差不多,Go语言支持一级函数,在具有一级函数的编程语言中，可以将函数分配给变量，然后从这些变量调用函数

声明变量时用`func`关键字

```go
package main

import (
	"fmt"
)

func test() {
	fmt.Println("test")
}
func testArg(val int) int {
	fmt.Println("testArg", val)
	return 0
}

//函数变量做另一个函数的参数
func testFunc(val int, myFunc func(int) int) {
	myFunc(val)
}

func main() {
	var myFunc func()
	var myFuncArg func(int) int

	myFunc = test
	myFuncArg = testArg

	myFunc()
	myFuncArg(3)

	testFunc(3, testArg)
}
```