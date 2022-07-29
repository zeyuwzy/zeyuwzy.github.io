---
title: Head First Go - 02条件和循环
date: 2022-07-22 08:19:11
tags:
    - Go
---

## 调用方法

`值.方法名(参数)` 类比调用另一个包的函数 `包.函数(参数)`

```go
package main

import (
	"fmt"
	"strings"
	"time"
)

func main() {
	var now time.Time = time.Now() //time.Now()返回time.Time类型
	var year int = now.Year() //time.Time有一个方法 Year 通过now变量调用
	fmt.Println(year)
	broken := "G# r#cks!"
	replacer := strings.NewReplacer("#", "o")
	fixed := replacer.Replace(broken)
	fmt.Println(fixed)
}
```
## 函数或方法多个返回值
Go中的函数或方法可以返回任意数量的返回值

1. 使用`_`空白标识符作为占位符
```go
package main

import (
        "bufio"
        "fmt"
        "os"
)

func main() {
        fmt.Print("Enter")
        reader := bufio.NewReader(os.Stdin)
        input, _ := reader.ReadString('\n')
        fmt.Println(input)
}
```

2. if语句进行错误处理
if的条件语句不需要() 但是要{}
```go
package main

import (
        "bufio"
        "fmt"
        "os"
        "log"
)

func main() {
        fmt.Print("Enter")
        reader := bufio.NewReader(os.Stdin)
        input, err := reader.ReadString('\n')
        if err != nil {
                log.Fatal(err)
        }
        fmt.Println(input)
}
```
## 要避免变量名与类型名、内置函数名、包名重复,否则会导致被变量名覆盖

## 变量作用域
和C差不多

## 短变量声明要求必须有一个变量是新的，新变量被视为声明，其他旧变量视为赋值

## 包的导入路径  
导入路径只是一个独特的字符串，用于标识包以及在导入语句中使 用的包。一旦导入了包，就可以通过其包名来引用它,但导入路径和包名称不必相同

Go语言不要求包名与其导入路径有任何关系。但按照惯例，导入路径的最后(或唯一)一段也用作包名

导入路径|包名
---|---  
fmt | fmt
math/rand | rand  

## 循环
for后面不用() 但是要{}

>for 初始化语句;条件表达式`(返回true false)`;标志语句 {
>
>}

初始化和标志(post)语句是可选的

`break`, `continue`和C一样


```go
// guess challenges players to guess a random number.
package main

import (
	"bufio"
	"fmt"
	"log"
	"math/rand"
	"os"
	"strconv"
	"strings"
	"time"
)

func main() {
	seconds := time.Now().Unix() //获取当前时间
	rand.Seed(seconds)//播种随机数生成器
	target := rand.Intn(100) + 1 //0 ~ 100的随机数
	fmt.Println("I've chosen a random number between 1 and 100.")
	fmt.Println("Can you guess it?")

	reader := bufio.NewReader(os.Stdin)
	success := false
	for guesses := 0; guesses < 10; guesses++ { //10次输入机会
		fmt.Println("You have", 10-guesses, "guesses left.")
		fmt.Print("Make a guess: ")
		input, err := reader.ReadString('\n')
		if err != nil {
			log.Fatal(err)
		}
		input = strings.TrimSpace(input) //删除换行
		guess, err := strconv.Atoi(input) //字符串转整数
		if err != nil {
			log.Fatal(err)
		}

		if guess < target {
			fmt.Println("Oops. Your guess was LOW.")
		} else if guess > target {
			fmt.Println("Oops. Your guess was HIGH.")
		} else {
			success = true
			fmt.Println("Good job! You guessed it!")
			break
		}
	}

	if !success {
		fmt.Println("Sorry, you didn't guess my number. It was:", target)
	}
}
```

## 其他 
字符串转数字 `strconv.ParseFloat`  
去除空白字符 `strings.TrimSpace`