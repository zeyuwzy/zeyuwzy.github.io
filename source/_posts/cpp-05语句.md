---
title: C++ 05语句
date: 2021-08-09 10:00:00
tags:
    - C++
---


## 简单语句

- **表达式语句**：一个表达式末尾加上分号，就变成了表达式语句
- **空语句**：只有一个单独的分号
- **复合语句（块）**：用花括号 `{}`包裹起来的语句和声明的序列。一个块就是一个作用域

## 条件语句

- **悬垂else**（dangling else）：用来描述在嵌套的`if else`语句中，如果`if`比`else`多时如何处理的问题。C++使用的方法是`else`匹配最近没有配对的`if`

## 迭代语句

- **while**：当不确定到底要迭代多少次时，使用 `while`循环比较合适，比如读取输入的内容
- **for**： `for`语句可以省略掉 `init-statement`， `condition`和 `expression`的任何一个；**甚至全部**
- **范围for**： `for (declaration: expression) statement`

## 跳转语句

- **break**：`break`语句负责终止离它最近的`while`、`do while`、`for`或者`switch`语句，并从这些语句之后的第一条语句开始继续执行
- **continue**：终止最近的循环中的当前迭代并立即开始下一次迭代。只能在`while`、`do while`、`for`循环的内部

## try语句块和异常处理

- **throw表达式**：异常检测部分使用 `throw`表达式来表示它遇到了无法处理的问题。我们说 `throw`引发 `raise`了异常
- **try语句块**：以 `try`关键词开始，以一个或多个 `catch`字句结束。 `try`语句块中的代码抛出的异常通常会被某个 `catch`捕获并处理。 `catch`子句也被称为**异常处理代码**
- **异常类**：用于在 `throw`表达式和相关的 `catch`子句之间传递异常的具体信息
- 只`throw`,而不进行异常捕捉,系统会调用`terminate`函数并终止当前程序的执行

|异常类型|描述|
|---|---|
|exception|最常见的问题|
|runtime_error|只有在运行时才能检测出的问题|
|range_error|运行时错误：生成的结果超出了有意义的值域范围|
|overflow_error|运行时错误：计算上溢|
|underflow_error|运行时错误：计算下溢|
|logic_error|程序逻辑错误|
|domain_error|逻辑错误：参数对应的结果值不存在|
|invalid_argument|逻辑错误：无效参数|
|length_error|逻辑错误：试图创建一个超出该类型最大长度的对象|
|out_of_range|逻辑错误：使用一个超出有效范围的值|


```c++
#include <iostream>
#include <stdexcept>

int main(int argc, char const *argv[])
{
    try {
        throw std::runtime_error("error");
    } catch (std::runtime_error err) {
        std:: cout << err.what() << std::endl;
    }
}
```