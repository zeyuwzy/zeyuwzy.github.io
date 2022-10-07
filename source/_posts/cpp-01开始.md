---
title: C++ 01开始
date: 2020-08-05 10:00:00
tags:
    - C++
---

## 输入输出
```c++
#include <iostream>
int main(){
    int a,b;    
    std::cin >> a >> b;
    std::cout << a + b << std::endl;
}
```
`<<`运算符接受两个运算对象:左侧的运算对象必须是一个`ostream`对象，右侧的运算对象是要打印的值。此运算符将给定的值写到给定的`ostream`对象中。输出运算符的计算结果就是其左侧运算对象。即计算结果就是我们写入给定值的那个`ostream`对象

`>>`接受一个`istream`作为其左侧运算对象，接受一个对象作为其右侧运算对象。它从给定的`istream`读入数据，并存入给定对象中。与输出运算符类似，输入运算符返回其左侧运算对象作为其计算结果

使用一个`istream`对象作为条件时，其效果是检测流的状态。如果流是有效的，即流未遇到错误，那么检测成功。当遇到文件结束符(end-of-file)，或遇到一个无效输入时（例如读入的值不是一个整数），istream对象的状态会变为无效。处于无效状态的istream对象会使条件变为假

```c++
//编写程序，从cin读取一组数，输出其和。
#include <iostream>

int main(int argc, char const *argv[])
{
    int sum = 0;
    for (int value = 0; std::cin >> value; ) //ctrl + d 停止
        sum += value;
    std::cout << sum << std::endl;
    return 0;
}
```

**endl**: 被称为**操纵符**（manipulator）的特殊值，效果是结束当前行，并将设备关联的缓冲区（buffer）中的内容刷到设备中。

Unix和Mac下:
- 键盘输入文件结束符：`ctrl+d`
- 获取命令状态`echo $?`
- 重定向 `./a.out < filea > fileb`

## 注释
- 单行注释： `//`
- 多行注释： `/**/`。编译器将`/*`和`*/`之间的内容都作为注释内容忽略。多行注释不能嵌套。