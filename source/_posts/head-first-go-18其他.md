---
title: Head First Go - 18其他
date: 2022-08-18 08:50:00
tags:
    - Go
---

## if语句初始化

    if 初始化; 条件 {

    }

```go
package main

import (
	"io/ioutil"
	"log"
)

func saveString(fileName string, str string) error {
	err := ioutil.WriteFile(fileName, []byte(str), 0600)
	return err
}

func main() {
	if err := saveString("english.txt", "Hello"); err != nil {
		log.Fatal(err)
	}
	if err := saveString("hindi.txt", "Namaste"); err != nil {
		log.Fatal(err)
	}
}
```

## switch语句
Go会在case代码末尾自动退出switch
如果你希望下一个case的代码也能运行，那么可以在一个case中使用fallthrough关键字

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func awardPrize() {
	switch rand.Intn(3) + 1 {
	case 1:
		fmt.Println("You win a cruise!")
	case 2:
		fmt.Println("You win a car!")
	case 3:
		fmt.Println("You win a goat!")
	default:
		panic("invalid door number")
	}
}

func main() {
	rand.Seed(time.Now().Unix())
	awardPrize()
}
```
## 符文编码
一个符文可以描述任意一个Unicode字符,Go支持将字符串转换为rune值的切片，并将符文切片转换回字符串。要使用部分字符串，应该将它们转换为rune值的切片，而不是byte值的切片。这样，你就不会意外地抓取符文的部分字节

Go允许你对字符串使用for...range循环，它一次处理一个符文，而不是一个字节。这是一种更安全的方法,保证每次获取的都是一个完整的字符

```go
package main

import (
	"fmt"
	"unicode/utf8"
)

func main() {

	asciiString := "ABCDE"
	utf8String := "БГДЖИ"

	fmt.Println("Byte lengths:")
	fmt.Println(len(asciiString))
	fmt.Println(len(utf8String))

	fmt.Println("Rune lengths:")
	fmt.Println(utf8.RuneCountInString(asciiString))
	fmt.Println(utf8.RuneCountInString(utf8String))

	fmt.Println("Slice operator on bytes:")
	asciiBytes := []byte(asciiString)
	utf8Bytes := []byte(utf8String)
	asciiBytesPartial := asciiBytes[3:]
	utf8BytesPartial := utf8Bytes[3:]
	fmt.Println(string(asciiBytesPartial))
	fmt.Println(string(utf8BytesPartial))

	fmt.Println("Slice operator on runes:")
	asciiRunes := []rune(asciiString)
	utf8Runes := []rune(utf8String)
	asciiRunesPartial := asciiRunes[3:]
	utf8RunesPartial := utf8Runes[3:]
	fmt.Println(string(asciiRunesPartial))
	fmt.Println(string(utf8RunesPartial))

	fmt.Println("Range on bytes:")
	for index, currentByte := range asciiBytes {
		fmt.Printf("%d: %s\n", index, string(currentByte))
	}
	for index, currentByte := range utf8Bytes {
		fmt.Printf("%d: %s\n", index, string(currentByte))
	}

	fmt.Println("Range on string:")
	for position, currentRune := range asciiString {
		fmt.Printf("%d: %s\n", position, string(currentRune))
	}
	for position, currentRune := range utf8String {
		fmt.Printf("%d: %s\n", position, string(currentRune))
	}
}

```

## 有缓冲的channel
当goroutine通过channel发送一个值时，该值被添加到缓冲区中。发送的goroutine将继续运行，而不被阻塞,直到缓冲区满。另一个goroutine从channel接收一个值时，它从缓冲区中提取最早添加的值

```go
package main

import (
	"fmt"
	"time"
)

func sendLetters(channel chan string) {
	time.Sleep(1 * time.Second)
	channel <- "a"
	time.Sleep(1 * time.Second)
	channel <- "b"
	time.Sleep(1 * time.Second)
	channel <- "c"
	time.Sleep(1 * time.Second)
	channel <- "d"
}

func main() {
	fmt.Println(time.Now())
	channel := make(chan string, 3)// 大小为3的缓冲区
	go sendLetters(channel)
	time.Sleep(5 * time.Second)
	fmt.Println(<-channel, time.Now())
	fmt.Println(<-channel, time.Now())
	fmt.Println(<-channel, time.Now())
	fmt.Println(<-channel, time.Now())
	fmt.Println(time.Now())
}
```