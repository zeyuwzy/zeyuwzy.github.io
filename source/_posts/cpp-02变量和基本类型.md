---
title: C++ 02变量和基本类型
date: 2020-08-06 10:00:00
tags:
    - C++
---

## 内置类型
| 类型 | 含义 | 最小尺寸|
|---|---|---|
| `bool` | 布尔类型  | 8bits |
| `char`| 字符 | 8bits |
| `wchar_t` | 宽字符 | 16bits |
| `char16_t` | Unicode字符 | 16bits |
| `char32_t` | Unicode字符 | 32bits |
| `short` | 短整型 | 16bits |
| `int` | 整型 | 16bits (在32位机器中是32bits) |
| `long` | 长整型 | 32bits |
| `long long` | 长整型 | 64bits （是在C++11中新定义的） |
| `float` | 单精度浮点数 | 6位有效数字 |
| `double` | 双精度浮点数 | 10位有效数字 |
| `long double` | 扩展精度浮点数 | 10位有效数字 |

```c++
int main(int argc, char const *argv[])
{
    bool a;
    char b;
    wchar_t c;
    char16_t d;
    char32_t e;
    short f;
    int g;
    long h;
    long long i;
    float j;
    double k;
    long double l;

    std::cout << "bool = " << sizeof(a) << std::endl;
    std::cout << "char = " << sizeof(b) << std::endl;
    std::cout << "wchar_t = " << sizeof(c) << std::endl;
    std::cout << "char16_t = " << sizeof(d) << std::endl;
    std::cout << "char32_t = " << sizeof(e) << std::endl;
    std::cout << "short = " << sizeof(f) << std::endl;
    std::cout << "int = " <<  sizeof(g) << std::endl;
    std::cout << "long = " << sizeof(h) << std::endl;
    std::cout << "long long = " << sizeof(i) << std::endl;
    std::cout << "float = " << sizeof(j) << std::endl;
    std::cout << "double = " << sizeof(k) << std::endl;
    std::cout << "long double = " << sizeof(l) << std::endl;

    return 0;
}
```

```
bool = 1
char = 1
wchar_t = 4
char16_t = 2
char32_t = 4
short = 2
int = 4
long = 8
long long = 8
float = 4
double = 8
long double = 16
```

## 如何选择类型
- 当明确知晓数值不可能为负时，选用无符号类型
- 使用int执行整数运算。在实际应用中，short常常显得太小而long一般和int有一样的尺寸。如果你的数值超过了int的表示范围，选用long long
- 在算术表达式中不要使用char或bool，只有在存放字符或布尔值时才使用它们。因为类型char在一些机器上是有符号的，而在另一些机器上又是无符号的
- 执行浮点数运算选用double

## 类型转换
切勿混用带符号类型和无符号类型,带符号数会自动地转换成无符号数
```c++
    int a = -1;
    unsigned b = 1;
    std::cout << a * b << std::endl;
```
```
4294967295
```

- 非布尔型赋给布尔型，初始值为0则结果为false，否则为true
- 布尔型赋给非布尔型，初始值为false结果为0，初始值为true结果为1

## 字面值常量
- 整型和浮点型
    ```c++
    11; //10
    011;//8
    0x11;//16
    3.14159e0; //浮点,指数用E或e
    1.5E0;      //浮点
    ```
- 字符和字符串
    ```c++
    'a';
    "abc";
    std::cout << "hello"
    "world" << std::endl;
    ```
    - 转义

        |描述|值|
        |---|---|
        |换行符|`\n`|
        |横向制表符|`\t`|
        |纵向制表符|`\v`|
        |反斜线|`\\`|
        |回车符|`\r`|
        |退格符|`\b`|
        |问号|`\?`|
        |进纸符|`\f`|
        |报警符|`\a`|
        |双引号|`\"`|
        |单引号|`\'`|

    - 泛化转义

        \x后紧跟1个或多个十六进制数字，或者\后紧跟1个、2个或3个八进制数字
        ```c++
        std::cout << '\115' << 'M'  << '\x4d' << std::endl;
        ```
        如果反斜线\后面跟着的八进制数字超过3个，只有前3个数字与\构成转义序列

- 布尔字面值 true，false
- 指针字面值 nullptr
- 指定字面值类型

|类型| 位置 | 值 |描述| 类型|
|---|---|---|---|---|
|字符和字符串字面值|前缀|u| Unicode 16字符| `char16_t`| 
|字符和字符串字面值|前缀|U| Unicode 32字符| `char32_t`| 
|字符和字符串字面值|前缀|L| 宽字符| `wchar_t`| 
|字符和字符串字面值|前缀|u8| UTF-8| `char`| 
|整形字面值|后缀|u / U| - | `unsigned`|
|整形字面值|后缀|l / L| - | `long`|
|整形字面值|后缀|ll / LL| - | `long long`|
|浮点型字面值|后缀|f / F| - | `float`|
|浮点型字面值|后缀|l / L| - | `long double`|
