---
title: Head First Go - 16html模板
date: 2022-08-16 08:30:00
tags:
    - Go
---

## template

Go提供了一个html/template包，它将从文件加载HTML，并为我们插入签名

html/template包基于text/template包。使用这两个包的方法几乎完全相同，但是html/template有一些额外的安全特性(自动转译模版中插入的html代码，避免恶意攻击)


```go
package main

import (
	"log"
	"os"
	"text/template"
)

func main() {
	/*
	在main中，我们调用text/template包的New函数，该函数返回一个指向新Template值的指针。
	然后我们调用Template上的Parse方法，并将字 符串"Here's my template!\n"传递给它。
	Parse使用字符串参数作为模板的文本。Parse返回模板和一个error值。我们将模板存储在tmpl变量中
	*/
	text := "Here's my template\n"
	tmpl, err := template.New("Text").Parse(text)
	if err != nil {
		log.Fatal(err)
	}

	err = tmpl.Execute(os.Stdout, nil) //将模版写入终端

	if err != nil {
		log.Fatal(err)
	}
}
```

`Template.Execute()`第一个参数传入一个`io.Writer`,只要满足`Write`接口就可以,例如`http.ResponseWriter`和`os.Stdout`都有`Write`方法

```go
package io // import "io"

type Writer interface {
	Write(p []byte) (n int, err error)
}

```

`Template`值的`Execute`方法的第二个参数允许你传入要插入到模板中的数据。它的类型是空接口，这意味着你可以传入任何类型的值。

### action插入
要在模板中插入数据，可以向模板文本添加action(操作)。action用双花括号`{{}}`表示。在双花括号中，指定要插入的数据或要模板执行的操作。每当模板遇到action时，它将计算其内容，并在action的位置将结果插入模板文本中。在一个操作中，你可以使用一个带有“dot”(点)的Execute方法来引用传递给它的数据值

### if action
模板中`{{if.}}`action与其对应的`{{end}}`标记之间的一段只有在条件为真时才会包含

### range action
在`{{range.}}`action与其对应的`{{end}}`标记之间的模板部分，将对数组、切片、映射或channel中收集的每个值进行重复。该部分中的任何操作也将被重复

### struct action 
在模版通过`{{.value}}`填充结构体的值

```go
package main

import (
	"log"
	"os"
	"text/template"
)

type mySt struct {
	Name string
	Age int
}

func execTemplate(text string, data interface{}) {
	tmpl, err := template.New("Text").Parse(text)
	if err != nil {
		log.Fatal(err)
	}

	err = tmpl.Execute(os.Stdout, data) 

	if err != nil {
		log.Fatal(err)
	}
}

func main() {
	execTemplate("hhh {{.}}\n", "abc")
	execTemplate("hhh {{if.}} run {{end}} hhh\n", true)
	execTemplate("hhh {{if.}} run {{end}} hhh\n", false)
	execTemplate("hhh {{range.}} run{{.}} {{end}}\n", []int{1, 2, 3})
	execTemplate("hhh {{.Name}}  {{.Age}}\n", mySt{ Name:"zeyu", Age:18})
}
```

## web程序响应html模版

```go
package main

import (
	"bufio"
	"fmt"
	"html/template"
	"log"
	"net/http"
	"os"
)

// Guestbook is a struct used in rendering view.html.
type Guestbook struct {
	SignatureCount int //签名总数
	Signatures     []string //签名内容
}

// check calls log.Fatal on any non-nil error.
func check(err error) {
	if err != nil {
		log.Fatal(err)
	}
}

// viewHandler reads guestbook signatures and displays them together
// with a count of all signatures.
func viewHandler(writer http.ResponseWriter, request *http.Request) {
	signatures := getStrings("signatures.txt") //逐行读文件，文件不存在返回空切片
	html, err := template.ParseFiles("view.html") //创建html模版
	check(err)
	guestbook := Guestbook{
		SignatureCount: len(signatures),
		Signatures:     signatures,
	}
	err = html.Execute(writer, guestbook) //将数据插入模版，并写回http.ResponseWriter
	check(err)
}

// newHandler displays a form to enter a signature.
func newHandler(writer http.ResponseWriter, request *http.Request) {
	html, err := template.ParseFiles("new.html")
	check(err)
	err = html.Execute(writer, nil)
	check(err)
}

// createHandler takes a POST request with a signature to add, and
// appends it to the signatures file.
func createHandler(writer http.ResponseWriter, request *http.Request) {
	signature := request.FormValue("signature") //获取html signature表单字段的值
	options := os.O_WRONLY | os.O_APPEND | os.O_CREATE
	file, err := os.OpenFile("signatures.txt", options, os.FileMode(0600))
	check(err)
	_, err = fmt.Fprintln(file, signature) //将值写入signatures.txt
	check(err)
	err = file.Close()
	check(err)
	http.Redirect(writer, request, "/guestbook", http.StatusFound) //将浏览器重定向到 /guestbook
}

// getStrings returns a slice of strings read from fileName, one
// string per line.
func getStrings(fileName string) []string {
	var lines []string
	file, err := os.Open(fileName)
	if os.IsNotExist(err) {
		return nil
	}
	check(err)
	defer file.Close()
	scanner := bufio.NewScanner(file)
	for scanner.Scan() {
		lines = append(lines, scanner.Text())
	}
	check(scanner.Err())
	return lines
}

func main() {
	http.HandleFunc("/guestbook", viewHandler) //查看签名列表的请求
	http.HandleFunc("/guestbook/new", newHandler) //获取html表单的请求
	http.HandleFunc("/guestbook/create", createHandler) //提交表单的请求
	err := http.ListenAndServe("localhost:8080", nil)
	log.Fatal(err)
}

```
[下载html链接](https://headfirstgo.com/source/head-first-go-ch16-code.zip)

## html模版
- `<h1>`  : 一级标题。通常以大的、粗体文本显示
- `<div>` : division元素。它本身不直接可见，但用于将页面划分为多个部分
- `<p>`   : 一段文字。我们将把每个签名作为一个单独的段落来处理
- `<a>`   : 代表“anchor”(锚)。创建一个链接


view.html
```html
<h1>Guestbook</h1> <!--页面顶部一级标题-->

<div>
	{{.SignatureCount}} total signatures - <!--这里插入签名数量-->
	<a href="/guestbook/new">Add Your Signature</a> <!--指向html表单路径的链接-->
</div>

<div>
	{{range .Signatures}}
		<p>{{.}}</p> <!--这里插入签名-->
	{{end}}
</div>

```

- `<form>`:该元素包含表单的所有其他组件
- 具有`type`属性为`"text"`的`<input>`:用户可以在其中输入字符串的文本字段。它的`name`属性将用于标记发送到服务器的数据中字段的值 (类似于映射键)
- 具有`type`属性为`"submit"`的`<input>`:创建一个按钮，用户可以单击该按钮来提交表单的数据
- `POST`:当浏览器需要向服务器添加一些数据时使用，通常是因为你提交了一个包含新数据的表单
- `GET`:当浏览器需要从服务器获取某些内容时使用，通常是因为你输入了URL或单击了链接。这可能是一个HTML页面、图像或其他资源

new.html

```html
<h1>Add a Signature</h1> <!--页面顶部一级标题-->
<!--表单一般使用form标签包裹-->
<form action="/guestbook/create" method="POST"> <!--提交转到/guestbook/create-->
  <div><input type="text" name="signature"></div> <!--文本字段，内容可以通过signature关键字访问-->
  <div><input type="submit"></div> <!--提交表单按钮-->
</form>

```