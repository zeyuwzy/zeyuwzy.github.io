---
title: C++ 02变量和基本类型
date: 2021-08-06 10:00:00
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

## 变量
**变量**提供一个**具名**的、可供程序操作的存储空间   

`C++`中**变量**和**对象**一般可以互换使用

### 变量定义（define）
- **定义形式**：类型说明符（type specifier） + 一个或多个变量名组成的列表。如`int sum = 0, value;`
- **初始化**（initialize）：对象在创建时获得了一个特定的值
  - **初始化不是赋值！**
  - 初始化 = 创建变量 + 赋予初始值
  - 赋值 = 擦除对象的当前值 + 用新值代替
  - **列表初始化**：使用花括号`{}`，如`int units_sold = {0};`
  - 默认初始化：定义时没有指定初始值会被默认初始化；**在函数体内部的内置类型变量将不会被初始化**
  - 建议初始化每一个内置类型的变量

### **变量声明**和**变量定义**
- 为了支持分离式编译，`C++`将声明和定义区分开。**声明**使得名字为程序所知。**定义**负责创建与名字关联的实体
- **extern**：只是说明变量定义在其他地方
  - 只声明而不定义： 在变量名前添加关键字 `extern`，如`extern int i;`。但如果包含了初始值，就变成了定义：`extern double pi = 3.14;`
  - 变量只能被定义一次，但是可以多次声明。定义只出现在一个文件中，其他文件使用该变量时需要对其声明

### 作用域
- 名字的**作用域**（namescope）用花括号分割`{}`
  - **第一次使用变量时再定义它**。
  - 嵌套的作用域
    - 同时存在全局和局部变量时，已定义局部变量的作用域中可用`::reused`显式访问全局变量reused。
    - **但是用到全局变量时，尽量不适用重名的局部变量**

## 左值和右值
- **左值**（l-value）**可以**出现在赋值语句的左边或者右边，比如变量；
- **右值**（r-value）**只能**出现在赋值语句的右边，比如常量。

## 引用
- 一般说的引用是指的左值引用
- **引用**：引用是一个对象的别名，引用类型引用（refer to）另外一种类型。如`int &refVal = val;`
- 引用必须初始化
- 引用和它的初始值是**绑定bind**在一起的，而**不是拷贝**。一旦定义就不能更改绑定为其他的对象

## 指针
> int *p;      //**指向int型对象**的指针

- 是一种 `"指向（point to）"`另外一种类型的复合类型。
- **定义**指针类型： `int *ip1;`，**从右向左读有助于阅读**，`ip1`是指向`int`类型的指针。
- 指针存放某个对象的**地址**。
- 获取对象的地址： `int i=42; int *p = &i;`。 `&`是**取地址符**。
- 指针的类型与所指向的对象类型必须一致（均为同一类型int、double等）
- 指针的值的四种状态：
  - 1.指向一个对象
  - 2.指向紧邻对象的下一个位置
  - 3.空指针
  - 4.无效指针
  - >**对无效指针的操作均会引发错误，第二种和第三种虽为有效的，但理论上是不被允许的**
- 指针访问对象： `cout << *p;`输出p指针所指对象的数据， `*`是**解引用符**
- 空指针不指向任何对象。使用`int *p=nullptr;`来使用空指针
- > 指针和引用的区别：引用本身并非一个对象，引用定义后就不能绑定到其他的对象了；指针并没有此限制，相当于变量一样使用
- > 赋值语句永远改变的是**左侧**的对象
- `void*`指针可以存放**任意**对象的地址。因无类型，仅操作内存空间，对所存对象无法访问
- 其他指针类型必须要与所指对象**严格匹配**
- 两个指针相减的类型是`ptrdiff_t`
- 建议：初始化所有指针
- `int* p1, p2;//*是对p1的修饰，所以p2还是int型`

## const限定符

- 动机：希望定义一些不能被改变值的变量。

### 初始化和const
- const对象**必须初始化**，且**不能被改变**。
- const变量默认不能被其他文件访问，非要访问，const变量不管是声明还是定义都添加extern关键字

### const的引用

- **reference to const**（对常量的引用）：指向const对象的引用，如 `const int ival=1; const int &refVal = ival;`，可以读取但不能修改`refVal`
- 初始化常量引用时允许用任意表达式(**包括非常量的对象**)作为初始值，只要该表达式的结果能转换成引用的类型即可
- **临时量**（temporary）对象：当编译器需要一个空间来暂存表达式的求值结果时，临时创建的一个未命名的对象
    ```c++
    double d = 3.14;
    const int &ref = d;
    //等价于
    const int tmp = d; //编译器生成一个临时量
    const int &ref = tmp;
    ```
- 对临时量的引用是非法行为

### 指针和const

- **pointer to const**（**指向常量的指针**）：不能用于改变其所指对象的值, 如 `const double pi = 3.14; const double *cptr = &pi;`
- **const pointer**：(**常量指针**)指针本身是常量，也就是说指针固定指向该对象，（存放在指针中的地址不变，地址所对应的那个对象值可以修改）如 `int i = 0; int *const ptr = &i;`

### 顶层const

- `顶层const`：指针本身是个常量
- `底层const`：指针指向的对象是个常量。拷贝时严格要求相同的底层const资格

### `constexpr`和常量表达式
- 常量表达式：指值不会改变，且在编译过程中就能得到计算结果的表达式
- `C++11`新标准规定，允许将变量声明为`constexpr`类型以便由编译器来验证变量的值是否是一个常量的表达式
- 定义一个constexpr引用，只能绑定在全局变量和局部静态变量上
    ```c++
        int g_num;

        int main(int argc, char const *argv[])
        {
            static int num = 5;  //定义一个constexpr引用，只能绑定在全局变量和局部静态变量上
            constexpr int &b = num;
            constexpr int &c = g_num;
        }
    ```
- constexpr指针的初始值必须是nullptr或者0，或者是存储于某个固定地址中的对象（全局变量和局部静态变量）
- constexpr声明中如果定义了一个指针，限定符constexpr仅对指针有效，与指针所指的对象无关(顶层const)
    ```c++
        constexpr int *p = nullptr; //p是常量指针
    ```
- constexpr修饰函数
  - 函数的所有形参及返回类型都是字面值类型
  - 函数体只有一条return语句
  - constexpr隐式地指定为内联函数
  - constexpr函数体还可以包含空语句、类型别名以及using声明，他们都没有执行其他操作，因此可以写入constexpr函数体
    ```c++
        constexpr int getSize() { return 15; }

        int main(int argc, char const *argv[])
        {
            constexpr int num = getSize();
        }
    ```
  - constexpr函数不一定会返回常量表达式,常量表达式以及constexpr才真正意义上定义了常量。以往的const我们可以理解为一种`Read Only`
    ```c++
    constexpr int getSize(size_t size) { return size; }

    int main(int argc, char const *argv[])
    {
        constexpr int num = getSize(15);


        int size = 15;
        int get = getSize(size); //返回的不是常量表达式,但是传给int可以编译成功

        constexpr int num = getSize(size); //错误
    }

    ```
  - constexpr构造函数
    - constexpr构造函数的函数体是空的，只能使用初始值列表进行所有成员的初始化。提供的初始值必须是一条常量表达式
    ```c++
        class Test
        {
        public:
            constexpr Test(int aa = 1, int bb = 2): m_a(aa), m_b(bb) { }
        private:
            int m_a;
            int m_b;
        };
    ```

## 别名
- 传统别名：使用**typedef**来定义类型的同义词。 `typedef double wages;`
- 新标准别名：别名声明（alias declaration）： `using SI = Sales_item;`（C++11）

```c++
// 对于复合类型（指针等）不能代回原式来进行理解
typedef char *pstring;  // pstring是char*的别名
const pstring cstr = 0; // 指向char的常量指针
// 如改写为const char *cstr = 0;不正确，为指向const char的指针

// 辅助理解（可代回后加括号）从内向外，从右向左
// const pstring cstr = 0;代回后const (char *) cstr = 0;
// const char *cstr = 0;即为(const char *) cstr = 0;
```

## auto类型说明符
- **auto**类型说明符：让编译器**自动推断类型**
- auto定义的变量必须有初始值
- 一条声明语句只能有一个数据类型，所以一个auto声明多个变量时只能相同的变量类型(包括复杂类型&和*)。`auto sz = 0, pi =3.14//错误`
- auto无法直接推断引用类型
- `int i = 0, &r = i; auto a = r;` 推断`a`的类型是`int`。
- 会忽略`顶层const`
- `const int ci = 1; auto b = ci;` 推断类型是`int` 
- `const auto f = ci;`，如果希望是顶层const需要自己加`const`

## decltype类型指示符
`decltype`并不会实际计算表达式的值，编译器分析表达式并得到它的类型,函数调用也算一种表达式

- decltype(变量) :直接返回变量的类型,包括顶层const和引用
- decltype(表达式) :表达式返回左值，得到该类型的左值引用；表达式返回右值，得到该类型
    ```c++
        int a = 5;
        decltype(a + 1) val; //a+1是表达式,返回右值 val是int

        decltype(a) val1;  //独作用于对象，没有使用对象的表达式的属性，而是直接获得了变量的类型 val1是int

        decltype((a)) val2; //获得变量作为表达式的类型，可以加一个括号, (a)是表达式,表达式返回左值a， val2的类型是int &
    ```
- decltype(函数名) :decltype会返回对应的函数类型，不会自动转换成相应的函数指针
    ```c++
    bool test(int a) {
        std::cout << a << std::endl;
        return true;
    }

    int main(int argc, char const *argv[])
    {
        decltype(test) *func = test; //通过decltype定义函数指针
        std::cout << func(15) << std::endl; 
    }
    ```

## 自定义数据结构
C++11新标准规定，可以为数据成员提供一个类内初始值（in-class initializer）。创建对象时，类内初始值将用于初始化数据成员。没有初始值的成员将被默认初始化

```c++
class MyClass 
{
private:
    int num = 5;

public:
    MyClass() {
        std::cout << num << std::endl;
    }
    ~MyClass() = default;
    int getNum() {
        return num;
    }

};
```

## 预处理
- **预处理器**（preprocessor）：确保头文件多次包含仍能安全工作
- 当预处理器看到`#include`标记时，会用指定的头文件内容代替`#include`
- **头文件保护符**（header guard）：头文件保护符依赖于预处理变量的状态：已定义和未定义
  - `#indef`已定义时为真
  - `#inndef`未定义时为真
  - 头文件保护符的名称需要唯一，且保持全部大写。养成良好习惯，不论是否该头文件被包含，要加保护符

```c++
#ifndef SALES_DATA_H  //SALES_DATA_H未定义时为真
#define SALES_DATA_H
strct Sale_data{
    ...
}
#endif
```