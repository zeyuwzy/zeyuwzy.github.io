---
title: Head First Go - 14自动化测试
date: 2022-08-14 21:00:00
tags:
    - Go
---

## go test
Go包含一个testing包，你可以用来为代码编写自动化测试，还有一个`go test`命令，你可以用来运行这些测试

需要新创建一个后缀名为`_test.go`的测试文件，go test工具查找以该后缀命名的文件


```go 
//join.go
package prose

import "strings"

func JoinWithCommas(phrases []string) string {
	if len(phrases) == 0 {
		return ""
	} else if len(phrases) == 1 {
		return phrases[0]
	} else if len(phrases) == 2 {
		return phrases[0] + " and " + phrases[1]
	} else {
		result := strings.Join(phrases[:len(phrases)-1], ", ")
		result += ", and "
		result += phrases[len(phrases)-1]
		return result
	}
}
```

```go
//join_test.go
package prose //要测试的包名,这个测试代码相当于和prose同属于一个包

import "testing" //导入testing包

type testData struct {
	list []string
	want string
}

//函数都以Test开头, go test 将检测这些函数
func TestJoinWithCommas(t *testing.T) {
	tests := []testData{
		testData{list: []string{}, want: ""},
		testData{list: []string{"apple"}, want: "apple"},
		testData{list: []string{"apple", "orange"}, want: "apple and orange"},
		testData{list: []string{"apple", "orange", "pear"}, want: "apple, orange, and pear"},
	}
	for _, test := range tests {
		got := JoinWithCommas(test.list)
		if got != test.want {
			t.Errorf("JoinWithCommas(%#v) = \"%s\", want \"%s\"", test.list, got, test.want) //一旦调用testing.T的Error或Errorf则表示失败
		}
	}
}

```

## 编写规则
- 你不需要将测试代码与正在测试的代码放在同一个包中，但是如果你想从包中访问未导出的类型或函数，则需要这样做
- 测试需要使用testing包中的类型，所以需要在每个测试文件的顶部导入该包
- 测试函数名应该以Test开头。(名字的其余部分可以是你想要的任何内容，但它应该以大写字母开头。)
- 测试函数应该接受单个参数:一个指向testing.T值的指针
- 你可以通过对testing.T值调用方法(例如Error)来报告测试失败。大多数方法都接受一个字符串，其中包含解释测试失败原因的信息

## 运行go test
```shell
go test (包导出路径)
go test -v # 包含详细信息
go test -run key  # 测试匹配到key的函数名
```
## 其他

`go run`,`go build`不会去包含那些测试代码 

_test.go文件中名字不以Test开头的函数不会由`go test`运行。它们可以被测试用作“helper”函数

表驱动测试是处理输入和预期输出的“表”的测试。它们将每一组输入传递给正在测试的代码，并检查代码的输出是否与预期值匹配