---
title: Head First Go - 17os.OpenFile
date: 2022-08-17 12:30:00
tags:
    - Go
---

## 二进制位表示
`%b`
```go
	fmt.Printf("%08b\n", 1)
	fmt.Printf("%08b\n", 127)
```
## 八进制位表示
`%o`
```go
	fmt.Printf("%08o\n", 9)
```

## 文件操作
文件操作基本延续系统调用,参考`go doc os`

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	file, err := os.OpenFile(os.Args[1], os.O_CREATE | os.O_RDWR, os.FileMode(0644))	
	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}

	len, err := file.Write([]byte("hhhhh"))
	if err != nil {
		fmt.Println(err)
		file.Close()
		os.Exit(1)
	}

	file.Close()
	fmt.Println(len)
}
```