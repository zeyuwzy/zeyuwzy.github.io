---
title: C++ Effective C++
date: 2021-08-24 10:00:00
tags:
    - C++
---

---
## 让自己习惯C++Accustoming Yourself to C++
---

## 条款01 视C++为一个语言联邦 (View C++as a federation of languages)
- 面向过程形式（procedural）
- 面向对象形式（object-oriented）
- 函数形式（functional）
- 泛型形式（generic）
- 元编程形式（metaprogramming)

## 条款02：尽量以const，enum，inline替换＃define (Prefer consts，enums，and inlines to＃defines)
- 对于单纯常量，最好以`const`对象或`enums`替换`＃defines`
- 对于形似函数的宏（macros），最好改用`inline`函数替换`＃defines`

## 条款03：尽可能使用const (Use const whenever possible)
- 将某些东西声明为`const`可帮助编译器侦测出错误用法。`const`可被施加于任何作用域内的对象、函数参数、函数返回类型、成员函数本体
- 编译器强制实施`bitwise constness`，但你编写程序时应该使用“概念上的常量性”（conceptual constness）,意味着你不应该将一个返回数据成员非常量引用的成员函数声明为`const`。虽然这个函数本身没有修改对象（即符合 bitwise constness），但调用者可以拿到这个非常量引用来修改对象内部的数据
- 当`const`和`non-const`成员函数有着实质等价的实现时，令`non-const`版本调用`const`版本可避免代码重复

## 条款04: 确定对象被使用前已先被初始化 (Make sure that objects are initialized before they're used)
- 为内置型对象进行手工初始化，因为C++不保证初始化它们
- 构造函数最好使用成员初值列（member initialization list），而不要在构造函数本体内使用赋值操作（assignment）。初值列列出的成员变量，其排列次序应该和它们在`class`中的声明次序相同
- 为免除“跨编译单元之初始化次序”问题，请以`local static`对象替换`non-local static`对象

---
## 构造/析构/赋值运算 Constructors，Destructors，and Assignment Operators
---

## 条款05：了解C++默默编写并调用哪些函数 (Know what functions C++silently writes and calls)
- 编译器可以暗自为`class`创建`default`构造函数、`copy`构造函数、`copy assignment`操作符，以及析构函数

## 条款06: 若不想使用编译器自动生成的函数，就该明确拒绝 (Explicitly disallow the use of compiler-generated functions you do not want)
- 为驳回编译器自动（暗自）提供的机能，可将相应的成员函数声明为`private`并且不予实现。使用像`Uncopyable`这样的`base class`也是一种做法

## 条款07: 为多态基类声明virtual析构函数 (Declare destructors virtual in polymorphic base classes)
- `polymorphic`（带多态性质的）`base classes`应该声明一个`virtual`析构函数。如果`class`带有任何`virtual`函数，它就应该拥有一个`virtual`析构函数
- `Classes`的设计目的如果不是作为`base classes`使用，或不是为了具备多态性（polymorphically），就不该声明`virtual`析构函数

## 条款08：别让异常逃离析构函数 (Prevent exceptions from leaving destructors)
- 析构函数绝对不要吐出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下它们（不传播）或结束程序
- 如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么`class`应该提供一个普通函数（而非在析构函数中）执行该操作

## 条款09：绝不在构造和析构过程中调用virtual函数 (Never call virtual functions during construction or destruction)
- 在`base`构造和析构期间不要调用`virtual`函数，因为这类调用从不下降至`derived class`（比起当前执行构造函数和析构函数的那层）

## 条款10：令operator=返回一个 reference to*this (Have assignment operators return a reference to*this)
- 令赋值（assignment）操作符返回一个reference to*this

## 条款11：在operator=中处理“自我赋值” (Handle assignment to self in operator=)
- 确保当对象自我赋值时`operator=`有良好行为。其中技术包括比较“来源对象”和“目标对象”的地址、精心周到的语句顺序、以及`copy-and-swap`
- 确定任何函数如果操作一个以上的对象，而其中多个对象是同一个对象时，其行为仍然正确

## 条款12：复制对象时勿忘其每一个成分 (Copy all parts of an object)
- `Copying`函数应该确保复制**对象内的所有成员变量**及**所有base class成分**
- 不要尝试以某个`copying`函数实现另一个`copying`函数。应该将共同机能放进第三个函数中，并由两个`coping`函数共同调用

---
## 资源管理 Resource Management
---

## 条款13：以对象管理资源 (Use objects to manage resources)
- 为防止资源泄漏，请使用`RAII`(Resource Acquisition Is Initialization)对象，它们在构造函数中获得资源并在析构函数中释放资源
- 两个常被使用的`RAII classes`分别是`tr1：：shared_ptr`和`auto_ptr`。前者通常是较佳选择，因为其`copy`行为比较直观。若选择`auto_ptr`，复制动作会使它（被复制物）指向`null`

## 条款14：在资源管理类中小心copying行为 (Think carefully about copying behavior in resource-managing classes)
- 复制`RAII`对象必须一并复制它所管理的资源，所以资源的`copying`行为决定`RAII`对象的`copying`行为
- 普遍而常见的`RAII class copying`行为是：抑制`copying`、施行引用计数法（reference counting）。不过其他行为也都可能被实现

## 条款15:资源管理类中提供对原始资源的访问 (Provide access to raw resources in resource-managing classes)
- `APIs`往往要求访问原始资源（raw resources），所以每一个`RAII class`应该提供一个“取得其所管理之资源”的办法
- 对原始资源的访问可能经由显式转换或隐式转换。一般而言显式转换比较安全，但隐式转换对客户比较方便

## 条款16：成对使用new和delete时要采取相同形式 (Use the same form in corresponding uses of new and delete)
- 如果你在`new`表达式中使用`[]`，必须在相应的`delete`表达式中也使用`[]`。如果你在`new`表达式中不使用`[]`，一定不要在相应的`delete`表达式中使用`[]`

## 条款17：以独立语句将newed对象置入智能指针 (Store newed objects in smart pointers in standalone statements)
- 以独立语句将`newed`对象存储于（置入）智能指针内。如果不这样做，一旦异常被抛出，有可能导致难以察觉的资源泄漏

---
## 设计与声明 Designs and Declarations
---

## 条款18：让接口容易被正确使用，不易被误用 (Make interfaces easy to use correctly and hard to use incorrectly)
- 好的接口很容易被正确使用，不容易被误用。你应该在你的所有接口中努力达成这些性质
- “促进正确使用”的办法包括接口的一致性，以及与内置类型的行为兼容
- “阻止误用”的办法包括建立新类型、限制类型上的操作，束缚对象值，以及消除客户的资源管理责任
- `tr1::shared_ptr`支持定制型删除器（custom deleter）。这可防范`DLL`问题，可被用来自动解除互斥锁（mutexes；见条款14）等等

## 条款19：设计class犹如设计type (Treat class design as type design)
- `Class`的设计就是`type`的设计。在定义一个新`type`之前，请确定你已经考虑过本条款覆盖的所有讨论主题

## 条款20：宁以pass-by-reference-to-const替换pass-by-value (Prefer pass-by-reference-to-const to pass-by-value)
- 尽量以`pass-by-reference-to-const`替换`pass-by-value`。前者通常比较高效，并可避免切割问题（slicing problem）
- 以上规则并不适用于内置类型，以及`STL`的迭代器和函数对象。对它们而言，`pass-by-value`往往比较适当

## 条款21：必须返回对象时，别妄想返回其reference (Don't try to return a reference when you must return an object)
- 绝不要返回`pointer`或`reference`指向一个`local stack`对象，或返回`reference`指向一个`heap-allocated`对象，或返回`pointer`或`reference`指向一个`local static`对象而有可能同时需要多个这样的对象。条款4已经为“在单线程环境中合理返回`reference`指向一个`local static`对象”提供了一份设计实例(单例)

## 条款22：将成员变量声明为private (Declare data members private)
- 切记将成员变量声明为`private`。这可赋予客户访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，并提供`class`作者以充分的实现弹性
- `protected`并不比`public`更具封装性

## 条款23：宁以non-member、non-friend替换member函数 (Prefer non-member non-friend functions to member functions)
- 宁可拿`non-member` `non-friend`函数替换`member`函数。这样做可以增加封装性、包裹弹性（packaging flexibility）和机能扩充性

## 条款24：若所有参数皆需类型转换，请为此采用non-member函数 (Declare non-member functions when type conversions should apply to all parameters)
- 如果你需要为某个函数的所有参数（包括被 this指针所指的那个隐喻参数）进行类型转换，那么这个函数必须是个`non-member`

## 条款25：考虑写出一个不抛异常的swap函数 (Consider support for a non-throwing swap)
- 当`std::swap`对你的类型效率不高时，提供一个`swap`成员函数，并确定这个函数不抛出异常
- 如果你提供一个`member swap`，也该提供一个`non-member swap`用来调用前者。对于`classes`（而非`templates`），也请特化`std::swap`
- 调用`swap`时应针对`std::swap`使用`using`声明式，然后调用`swap`并且不带任何“命名空间资格修饰”
- 为“用户定义类型”进行`std templates`全特化是好的，但千万不要尝试在`std`内加入某些对`std`而言全新的东西

--- 
## 实现 Implementations
---

## 条款26：尽可能延后变量定义式的出现时间 (Postpone variable definitions as long as possible)
- 尽可能延后变量定义式的出现。这样做可增加程序的清晰度并改善程序效率

## 条款27：尽量少做转型动作 (Minimize casting)
- 如果可以，尽量避免转型，特别是在注重效率的代码中避免`dynamic_casts`如果有个设计需要转型动作，试着发展无需转型的替代设计
- 如果转型是必要的，试着将它隐藏于某个函数背后。客户随后可以调用该函数，而不需将转型放进他们自己的代码内
- 宁可使用`C++-style`（新式）转型，不要使用旧式转型。前者很容易辨识出来，而且也比较有着分门别类的职掌

## 条款28：避免返回handles指向对象内部成分 (Avoid returning"handles"to object internals)
- 避免返回`handles`（包括`references`、指针、迭代器）指向对象内部。遵守这个条款可增加封装性，帮助`const`成员函数的行为像个`const`，并将发生“虚吊号码牌”（dangling handles）的可能性降至最低(声明了`cosnt`成员函数返回一个成员变量的引用)

## 条款29：为“异常安全”而努力是值得的 (Strive for exception-safe code)
- 异常安全函数（Exception-safe functions）即使发生异常也不会泄漏资源或允许任何数据结构败坏。这样的函数区分为三种可能的保证：基本型、强烈型、不抛异常型
- “强烈保证”往往能够以`copy-and-swap`实现出来，但“强烈保证”并非对所有函数都可实现或具备现实意义
- 函数提供的“异常安全保证”通常最高只等于其所调用之各个函数的“异常安全保证”中的最弱者

## 条款30：透彻了解inlining的里里外外 (Understand the ins and outs of inlining)
- 将大多数`inlining`限制在小型、被频繁调用的函数身上。这可使日后的调试过程和二进制升级（binary upgradability）更容易，也可使潜在的代码膨胀问题最小化，使程序的速度提升机会最大化
- 不要只因为`function templates`出现在头文件，就将它们声明为`inline`

## 条款31：将文件间的编译依存关系降至最低 (Minimize compilation dependencies between files)
- 支持“编译依存性最小化”的一般构想是：相依于声明式，不要相依于定义式。基于此构想的两个手段是`Handle classes`和`Interface classes`
- 程序库头文件应该以“完全且仅有声明式”（full and declaration-only forms）的形式存在。这种做法不论是否涉及`templates`都适用

---
## 继承与面向对象设计 Inheritance and Object-Oriented Design
---

## 条款32：确定你的public继承塑模出is-a关系 (Make sure public inheritance models"is-a.")
- `public`继承”意味`is-a`。适用于`base classes`身上的每一件事情一定也适用于`derived classes`身上，因为每一个`derived class`对象也都是一个`base class`对象

## 条款33：避免遮掩继承而来的名称 (Avoid hiding inherited names)
- `derived classes`内的名称会遮掩`base classes`内的名称。在`public`继承下从来没有人希望如此
- 为了让被遮掩的名称再见天日，可使用`using`声明式或转交函数（forwarding functions）

## 条款34：区分接口继承和实现继承 (Differentiate between inheritance of interface and inheritance of implementation)
- 接口继承和实现继承不同。在`public`继承之下，`derived classes`总是继承`base class`的接口
- `pure virtual`函数只具体指定接口继承
- 简朴的（非纯）`impure virtual`函数具体指定接口继承及缺省实现继承
- `non-virtual`函数具体指定接口继承以及强制性实现继承

## 条款35：考虑virtual函数以外的其他选择 (Consider alternatives to virtual functions)
- `virtual`函数的替代方案包括`NVI`(non-virtual interface)手法及`Strategy`设计模式的多种形式。`NVI`手法自身是一个特殊形式的`Template Method`设计模式
- 将机能从成员函数移到`class`外部函数，带来的一个缺点是，非成员函数无法访问`class`的`non-public`成员
- `tr1：：function`对象的行为就像一般函数指针。这样的对象可接纳`与给定之目标签名式（target signature）兼容`的所有可调用物（callable entities）

## 条款36：绝不重新定义继承而来的non-virtual函数 Never redefine an inherited non-virtual function
- 绝对不要重新定义继承而来的`non-virtual`函数

## 条款37：绝不重新定义继承而来的缺省参数值 (Never redefine a function's inherited default parameter value)
- 绝对不要重新定义一个继承而来的缺省参数值，因为缺省参数值都是静态绑定，而`virtual`函数——你唯一应该覆写的东西——却是动态绑定

## 条款38：通过复合塑模出has-a或“根据某物实现出” (Model"has-a"or"is-implemented-in-terms-of"through composition)
- 复合（composition）的意义和`public`继承完全不同。
- 在应用域（application domain），复合意味`has-a`（有一个）。在实现域（implementation domain），复合意味`is-implemented-in-terms-of`（根据某物实现出)

## 条款39：明智而审慎地使用private继承 (Use private inheritance judiciously)
- `Private`继承意味`is-implemented-in-terms of`（根据某物实现出）。它通常比复合（composition）的级别低。但是当`derived class`需要访问`protected base class`的成员，或需要重新定义继承而来的`virtual`函数时，这么设计是合理的
- 和复合（composition）不同，`private`继承可以造成`empty base`最优化。这对致力于“对象尺寸最小化”的程序库开发者而言，可能很重要

## 条款40：明智而审慎地使用多重继承 (Use multiple inheritance judiciously)
- 多重继承比单一继承复杂。它可能导致新的歧义性，以及对`virtual`继承的需要
- `virtual`继承会增加大小、速度、初始化（及赋值）复杂度等等成本。如果`virtual base classes`不带任何数据，将是最具实用价值的情况
- 多重继承的确有正当用途。其中一个情节涉及`public继承某个Interface class`和`private继承某个协助实现的class`的两相组合

---
## 模板与泛型编程 Templates and Generic Programming
---

## 条款41：了解隐式接口和编译期多态 (Understand implicit interfaces and compile-time polymorphism)
- `classes`和`templates`都支持接口（interfaces）和多态（polymorphism）
- 对`classes`而言接口是显式的（explicit），以函数签名为中心。多态则是通过`virtual`函数发生于运行期
- 对`template`参数而言，接口是隐式的（implicit），奠基于有效表达式。多态则是通过`template`具现化和函数重载解析（function overloading resolution）发生于编译期

## 条款42：了解typename的双重意义 (Understand the two meanings of typename)
- 声明`template`参数时，前缀关键字`class`和`typename`可互换
- 请使用关键字`typename`标识嵌套从属类型名称；但不得在`base class lists`（基类列）或`member initialization list`（成员初值列）内以它作为`base class`修饰符 

## 条款43：学习处理模板化基类内的名称 (Know how to access names in templatized base classes)
- 可在`derived class templates`内通过 "this->gt；" 指涉`base class templates`内的成员名称，或藉由一个明白写出的“base class资格修饰符”完成

## 条款44: 将与参数无关的代码抽离templates (Factor parameter-independent code out of templates)
- `Templates`生成多个`classes`和多个函数，所以任何`template`代码都不该与某个造成膨胀的`template`参数产生相依关系
- 因非类型模板参数（non-type template parameters）而造成的代码膨胀，往往可消除，做法是以函数参数或`class`成员变量替换`template`参数
- 因类型参数（type parameters）而造成的代码膨胀，往往可降低，做法是让带有完全相同二进制表述（binary representations）的具现类型（instantiation types）共享实现码

## 条款45: 运用成员函数模板接受所有兼容类型 (Use member function templates to accept"all compatible types.")
- 请使用`member function templates`（成员函数模板）生成“可接受所有兼容类型”的函数
- 如果你声明`member templates`用于“泛化copy构造”或“泛化assignment操作”，你还是需要声明正常的`copy`构造函数和`copy assignment`操作符

## 条款46：需要类型转换时请为模板定义非成员函数 (Define non-member functions inside templates when type conversions are desired)
- 当我们编写一个`class template`，而它所提供之`与此template相关的`函数支持`所有参数之隐式类型转换`时，请将那些函数定义为`class template 内部的friend函数`

## 条款47：请使用traits classes表现类型信息 (Use traits classes for information about types)
- `Traits classes`使得“类型相关信息”在编译期可用。它们以`templates`和`templates特化`完成实现
- 整合重载技术（overloading）后,`traits classes`有可能在编译期对类型执行`if...else`测试

## 条款48：认识template元编程 (Be aware of template metaprogramming.)
- `Template metaprogramming`（TMP，模板元编程）可将工作由运行期移往编译期，因而得以实现早期错误侦测和更高的执行效率
- `TMP`可被用来生成**基于政策选择组合**(based on combinations of policy choices）的客户定制代码，也可用来避免生成对某些特殊类型并不适合的代码

---
## 定制new和delete Customizing new and delete
---

## 条款49：了解new-handler的行为 (Understand the behavior of the new-handler)
- `set_new_handler`允许客户指定一个函数，在内存分配无法获得满足时被调用
- `Nothrow new`是一个颇为局限的工具，因为它只适用于内存分配；后继的构造函数调用还是可能抛出异常

## 条款50：了解new和delete的合理替换时机 (Understand when it makes sense to replace new and delete)
- 有许多理由需要写个自定的`new`和`delete`，包括改善效能、对`heap`运用错误进行调试、收集`heap`使用信息

## 条款51：编写new和delete时需固守常规 (Adhere to convention when writing new and delete)
- `operator new`应该内含一个无穷循环，并在其中尝试分配内存，如果它无法满足内存需求，就该调用`new-handler`。它也应该有能力处理`0 bytes`申请。`Class`专属版本则还应该处理“比正确大小更大的（错误）申请”。
- `operator delete`应该在收到`null`指针时不做任何事。`Class`专属版本则还应该处理“比正确大小更大的（错误）申请”

## 条款52：写了placement new也要写placement delete (Write placement delete if you write placement new)
- 当你写一个`placement operator new`，请确定也写出了对应的`placementoperator delete`。如果没有这样做，你的程序可能会发生隐微而时断时续的内存泄漏
- 当你声明`placement new`和`placement delete`，请确定不要无意识（非故意）地遮掩了它们的正常版本

---
## 杂项讨论 Miscellany
---

## 条款53：不要轻忽编译器的警告 (Pay attention to compiler warnings)
- 严肃对待编译器发出的警告信息。努力在你的编译器的最高（最严苛）警告级别下争取“无任何警告”的荣誉
- 不要过度倚赖编译器的报警能力，因为不同的编译器对待事情的态度并不相同。一旦移植到另一个编译器上，你原本倚赖的警告信息有可能消失

## 条款54：让自己熟悉包括TR1在内的标准程序库 (Familiarize yourself with the standard library，including TR1)
## 条款55：让自己熟悉Boost (Familiarize yourself with Boost)