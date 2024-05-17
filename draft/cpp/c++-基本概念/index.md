# [未完成] C++ 基本概念


## C 风格数组退化

C 语言传递数组时，数组类型会退化（decay）成指针类型，同时丢失长度信息。

> C 风格的数组在大多数情况下退化为指针，认为数组 = 指针是常见的谬误。数组是一个元素序列，而指针只保存一个地址。

```cpp
// [] 中的长度是无效的，所以不需要填
void printElementZero(const int arr[], int length);
// 等同于
void printElementZero(const int* arr, int length);
```

## 字符串

### C 风格字符串

C 风格的两种字符串定义方式：

```cpp
const char name[] { "Alex" };        // case 1
const char* const color{ "Orange" }; // case 2
```

case 1 中，"Alex\0" 先分配在只读段，然后 char 类型长度为 5 的数组，再使用 "Alex\0" 填充。所以存在两个副本。

case 2 中，"Orange" 被放入只读段，使用字符串的地址初始化指针。

auto 推导字符串：

```cpp
auto s1{ "Alex" };  // type deduced as const char*
auto* s2{ "Alex" }; // type deduced as const char*
auto& s3{ "Alex" }; // type deduced as const char(&)[5]
```

## 操作符

### sizeof

语法：

```cpp
sizeof( 类型 )	(1)	
sizeof 表达式	(2)	
```

1. sizeof a + b 会被解释成 sizeof(a) + b。
2. 当应用于类类型时，结果是该类的完整对象所占据的字节数，**包括这种对象放入数组时所需的任何额外填充**。
3. sizeof 的结果始终非零，即使应用于空类，一般是 1。

    ```cpp
    class A {};                      // sizeof(A) = 1;
    class B : public A { int x; };   // sizeof(B) = 4;
    class C {                        // sizeof(C) = 8;
        A a;
        B b;
    };
    ```

    > 空基类优化：为保证同一类型的不同对象地址始终有别，要求任何对象或成员子对象的大小至少为 1，即使该类型是空的类类型。空类的派生类可以允许空的基类子对象大小为零。

4. 当应用于某个表达式时，sizeof **不对表达式进行求值**，即便表达式代表多态对象，它的结果也是该表达式的静态类型的大小。

    ```cpp
    int a = 1;
    std::size_t b = sizeof ++a;
    std::cout << a << " " << b << "\n";
    ```

## CV 限定符

- const —— 定义类型为常量类型，reade only。
- volatile —— 定义类型为易变类型。

> 1. 直接修改 const 修饰的变量会编译失败，通过指针或引用修改则行为未定义
> 2. 一般需要关键字 volatile 的场合：
>    - Memory-mapped peripheral registers
>    - Global variables modified by an interrupt service routine
>    - Global variables within a multi-threaded application

不应该将二者理解成反义词：

1. const 保证被修饰的变量不会被程序（程度员）修改，但不负责源码以外被修改的情况。
2. volatile 说明变量可能被硬件改变，告知编译器不要优化此处

```cpp
uint8_t volatile *const pReg = (uint8_t*)0x40000000; 
uint8_t const volatile *const pReg = (uint8_t*)0x40000000; 
```

第一句说明常量指针 pReg 指向的 uint8_t 类型值，即地址 0x40000000 处的值可能被硬件修改。第二句说明，0x40000000 处的值可能被硬件修改，但不允许程序员修改。

## auto / decltype 

auto 自动类型推导：

- auto 总是优先推导为值类型
- 可以结合 cv 限定符，* 和 & 得到新类型
- auto&& 可被推导成左值引用或右值引用类型

```cpp
auto x = 10L;
auto& x1 = x;
auto* x2 = &x;
const auto& x3 = x;
auto x4 = &x3;
auto&& x5 = x;
```

decltype 不仅能推导出值类型，还可以推导出引用类型。

| 语法 | 说明 |
| --- | --- |
| decltype ( 实体 )	| (1)	(C++11 起) |
| decltype ( 表达式 ) | (2)	(C++11 起) |

cppref: 注意如果对象的名字带有括号，那么它会被当做普通的左值表达式，因此 decltype(x) 和 decltype((x)) 通常是不同的类型。

```cpp
int z = 0;
decltype(z)& z2 = z;  // z2 推导为 int&
decltype(z2) z6 = z2; // z6 推导为 int&
auto z7 = z2;         // z7 推导为 int
```

C++14 后，如果占位类型说明符是 decltype(auto)，那么推导出的类型是 decltype(expr)。

```cpp
int y = 10;
decltype(auto) y1 = (y);  // 推导为 int&, 因为（expr）是引用类型。
decltype(auto) y2 = &y;   // 推导为 int*
decltype(auto) y3 = y1;   // 推导为 int&
```

## 引用

C 语言中的 NULL 可以被定义为 0、0L、`((void*)0)` 或者其他，当空指针仍然使用 NULL 时，可能会出现问题：

```cpp
void print(int x) {
    std::cout << "print(int): " << x << std::endl;
}
void print(int* ptr) {
    std::cout << "print(int*): " << (ptr ? "non-null\n" : "null\n");
}
int main()
{
    print(0);
    print(NULL);  // clang / gcc 下编译错误，MSVC print(int)
    print(nullptr);
    return 0;
}
```

## 初始化

> C++ 中，初始化不等于赋值 -- C++ primer

![Cpp_initialization](/CPP/Cpp_initialization.png)

总体而言，C++ 的初始器 (initializer) 分为：

1. 复制初始化 ` = expression`
2. 直接初始化 ` ( ) `
3. 列表初始化 ` = { }`

复制初始化可能调用非 explicit 的构造函数，拷贝构造函数，或者是移动构造函数。

## 函数重载

// TODO:
重载函数规则：

注意，重载函数之间不一定需要完全独立，可以通过委托已经实现的同名函数进行重载，比如：

```cpp
void printMessage(const std::string& message) {
    std::cout << "Message: " << message << std::endl;
}

void printMessage(int number) {
    printMessage(std::to_string(number)); // 嵌套调用另一个重载函数
}

printMessage("Hello, World!");
printMessage(42);
```

这种手段可以增强复用性。

## 枚举

### C-style enum

普通枚举，将被隐式转换成整数，类型不安全。

```cpp
enum Color {  red,green,blue,  }; 
```

枚举名称非必须，但现代 C++ 应避免未命名的枚举。

可以指定枚举的基础类型，默认一般是 int。

```cpp
enum Color : std::int8_t {  black,red,blue, };
```

### enum class

枚举类不属于类类型，需要作用域解析运算符访问，能避免和其他枚举定义重名的问题，不会隐式转换成整数，类型安全。

```cpp
enum class Color {  red,green,blue,  }; 
```

C++ 20 起 using 可用于枚举：

```cpp
enum class Status{open, progress, done = 9};
void print(Status s) {
    switch (s)
    {
    using enum Status;
    // 或者：using Status::open, Status::done, Status::progress;
    case open:
        std::cout << "open";
        break;
    case progress:
        std::cout << "in progress";
        break;
    case done:
        std::cout << "done";
        break;
    }
}
```

## 函数指针和 std::function

TODO:

普通函数指针：

使用 typedef 作为声明标识符，为函数指针起别名，实现回调函数：

```cpp
// 函数指针类型，下面的语句定义的名字是 Button_Call，代表 void ()(void*) 类型的函数
typedef void (*Button_Call)(void*);
// 回调函数实现
void myButtonPressed(void* data) {
    printf("Button pressed data: %d\n", *((int*)data));
}
// 注册回调
void registerButtonPressCallback(Button_Call func, void * data) {
    func(data);
}

int main(int argc, char *argv[])
{
   int somedata = 42;
   registerButtonPressCallback(myButtonPressed, &somedata);
   return 0;
}
```

> 技巧：using + decltype 完成函数指针获取

```cpp
int add(int a, int b) { return a + b; }

using add_ptr_t = decltype(&add);
add_ptr_t my_add_ptr = add;
```

## 表达式

表达式是运算符和它们的操作数的序列，指定一项计算。

### 初等表达式（Primary expressions）

包括：

- this
- 字面量（例如 2 或 "Hello, world"）
- 标识表达式
- lambda 表达式 (C++11 起)
- 折叠表达式 (C++17 起)
- requires 表达式 (C++20 起)

任何带括号表达式也被归类为初等表达式。

### 常量表达式

constexpr 和 const 存在结合使用的场景，例如下面的代码会出现警告：

```cpp
constexpr char *msg = "Hello, world!";
> warning: ISO C++ forbids converting a string constant to ‘char*’ [-Wwrite-strings]
```

constexpr 会隐式在顶层添加一个 const 限定符：

```cpp
constexpr T a = xxx;
// 等价于：
T const a = xxx;
```

因此字符串本身的常量性未修饰。应该修改为：

```cpp
constexpr const char *p = "Hello, world!";
```

思考下面的例子：

```cpp
/*
 *           +----------+ +-----+ +-----+     
 * +-------- |const char|*|const|*|const| p  
 * |         +----------+ +--+--+ +--+--+     
 * |                         |       |        
 * |    +--------------------+       |        
 * |    |                            |        
 * |    |           +----------------+        
 * |    |           |                         
 * |    |           v      +-----------------+
 * |    |          ptr  -->|     0x1000      |
 * |    |                  +-----------------+
 * |    |                                     
 * |    |                  +-----------------+
 * |    +-------> 0x1000-->|     0x2000      |
 * |                       +-----------------+
 * |                                          
 * |                       +-----------------+
 * +------------> 0x2000-->|       'A'       |
 *                         +-----------------+
 */

constexpr char a = 'A';
constexpr const char* p = &a;
constexpr const char* const * pp = &p;

const char b = 'A';
const char* const q = &a;
const char* const * const qq = &p;
```

### 完整表达式（Full-Expression）

1. 初始化
2. 常量表达式
3. 立即调用（consteval）
4. 未求值的操作数
5. 不是子表达式的表达式

> 右值对象在创建它的完整表达式的末尾被销毁。对右值对象成员的任何引用都将在该点悬空。对右值对象成员的引用只能在创建右值对象的完整表达式中安全使用。

下面的 case2 属于未定义行为。

```cpp
class Employee
{
	std::string m_name{};

public:
	void setName(std::string_view name) { m_name = name; }
	const std::string& getName() const { return m_name; } //  getter returns by const reference
};

// createEmployee() returns an Employee by value (which means the returned value is an rvalue)
Employee createEmployee(std::string_view name)
{
	Employee e;
	e.setName(name);
	return e;
}

int main()
{
	// Case 1: okay: use returned reference to member of rvalue class object in same expression
	std::cout << createEmployee("Frank").getName();

	// Case 2: bad: save returned reference to member of rvalue class object for use later
	const std::string& ref { createEmployee("Garbo").getName() }; // reference becomes dangling when return value of createEmployee() is destroyed
	std::cout << ref; // undefined behavior

	// Case 3: okay: copy referenced value to local variable for use later
	std::string val { createEmployee("Hans").getName() }; // makes copy of referenced member
	std::cout << val; // okay: val is independent of referenced member

	return 0;
}
```

## 仿函数（functor）

通过在类内对 `()` 运算符进行重载可以得到仿函数，也称函数对象。仿函数可以拥有初始状态，一般通过 class 定义私有成员，在声明对象时初始化，私有成员的状态成为仿函数的初始状态。带状态的仿函数与 lambda 类似，仿函数可以用于实现 lambda 表达式（C++11）。

```cpp
class Tax
{
private:
    float rate;
    int base;
public:
    Tax(float r, int b)
        :rate(r), base(b)
    { }
    float operator()(float money){ return (money - base)*rate; }
};

int main(int argc, char *argv[])
{
    // <-- functor
    Tax high(0.40, 30000);
    Tax middle(0.25, 20000);

    cout << "tax over 3w: " << high(37500) << endl;
    cout << "tax over 2w: " << middle(27500) << endl;
}
```

## lambda 

## 偏函数

参数较多的函数，某些场合我们会固定某些参数，只用某种特化的可调用对象。使用 `std::bind` 固定某些参数，其余参数使用用 `std::placeholders` 占位符占位即可实现。

```cpp
#include <functional>
...
// 函数：将任意进制的数转换为十进制数
int anyBaseToDecimal(const std::string& num, int base) {
    int decimalNum = 0;
    int power = 0;
    for (int i = num.size() - 1; i >= 0; --i) {
        int digit = isdigit(num[i]) ? num[i] - '0' : num[i] - 'A' + 10;
        decimalNum += digit * pow(base, power);
        power++;
    }
    return decimalNum;
}

// 使用 std::bind 和 std::placeholders 构造特定进制转换的偏函数
int main() {std:
    // 绑定转换为二进制的偏函数
    auto binaryConverter = std::bind(anyBaseToDecimal, std::placeholders::_1, 2);
    std::cout << "Binary to Decimal: " << binaryConverter("1010") << std::endl;
    // 绑定转换为十六进制的偏函数
    auto hexConverter = std::bind(anyBaseToDecimal, :placeholders::_1, 16);
    std::cout << "Hexadecimal to Decimal: " << hexConverter("1A") << std::endl;
    return 0;
}
```

## 智能指针

## new 操作符

### placement new 

允许在被指定位置构造对象，可以在栈、静态区等位置构建。

```cpp
struct MyObject {
    int value;
    MyObject(int value) : value(value) 
    ~MyObject() 
};

// Define a static buffer
alignas(MyObject) static uint8_t buf[sizeof(MyObject)];

int main() {
    const MyObject* obj = new(buf) MyObject(42); // Placement new to construct object in buf
    // Use the object
    // For example, calling a member function
    obj->~MyObject(); // Explicitly call the destructor
    return 0;
}
```

## 转换（Conversions）

C++ 的四种类型转换：

- static_cast 转换一个类型为另一相关类型
- dynamic_cast 在继承层级中转换
- const_cast 添加或移除 cv 限定符
- reinterpret_cast 转换类型到无关类型

相比之下，C++ 的类型转换每次只转换类型的一个方面，C 风格的转换行为难以控制。

## 字面量（Literals）

用以表现嵌入到源代码中的常量值的记号，主要有整数、字符、浮点、字符串、bool、nullptr 和用户定义字面量。

### 分隔符（C++14）

数字之间可使用单引号分隔符，方便阅读，编译时会忽略：

```cpp
int a = 0b0001'1001'0001;
double b = 3.141'5926'5358'9793'2385;
```

类似于 Python 3.6 中数字的下划线：

```python
a = 1_000_000
b = 3.68_796
```

### 用户定义字面量

## initializer_list

`std::initializer_list<T>` 类型的对象是轻量代理对象，在编译时创建 const T 的对象数组，提供迭代器和 size()。例如：

```cpp
#include <iostream>
int main() {
    for (auto x : {"hello", "coding", "world"})
        ;
}
```

可通过 cppinsights 查看实际执行：

```cpp
#include <iostream>

int main()
{
  {
    const char *const __list4_19[3]{"hello", "coding", "world"};
    std::initializer_list<const char *> && __range1 = std::initializer_list<const char *>{__list4_19, 3};
    const char *const * __begin1 = __range1.begin();
    const char *const * __end1 = __range1.end();
    for(; __begin1 != __end1; ++__begin1) {
      const char * x = *__begin1;
    }
  }
  return 0;
}
```

注意：

1. std::initializer_list 是“视图”类型，主要用于传递函数参数，如果用作返回值则面临生命周期的问题。
2. 编译时创建的 const T 分配在栈空间或者只读内存上。

## 值类别

## 异常

一般认为，重要的构造函数、复制构造函数和移动构造函数应该尽量声明为 noexcept，析构函数则必须保证不抛出异常。

## 参考

1. [C-style array decay](https://www.learncpp.com/cpp-tutorial/c-style-array-decay/)
2. [C-style string symbolic constants](https://www.learncpp.com/cpp-tutorial/c-style-string-symbolic-constants/)
3. [Microcontroller Embedded C Programming Lecture 143: Usage of const and volatile together](https://fastbitlab.com/microcontroller-embedded-c-programming-lecture-143-usage-of-const-and-volatile-together/)
4. [What does a variable that is both const and volatile mean?](https://www.quora.com/What-does-a-variable-that-is-both-const-and-volatile-mean)
5. [Pass by address](https://www.learncpp.com/cpp-tutorial/pass-by-address-part-2/)
6. [复制初始化](https://zh.cppreference.com/w/cpp/language/copy_initialization)
7. [Member functions returning references to data members](https://www.learncpp.com/cpp-tutorial/member-functions-returning-references-to-data-members/)
8. [克服边界：在 C++ 中兼容 C 语言风格的函数指针回调的方式](https://zhuanlan.zhihu.com/p/689415121)
9. [使用 constexpr 时遇到的小坑](https://www.cnblogs.com/apocelipes/p/14769971.html)
10. [C++11 lambda 表达式与仿函数](https://www.cnblogs.com/zyk1113/p/13217312.html)
11. [仿函数](https://cui-jiacai.gitbook.io/c++-stl-tutorial/fang-han-shu-functor#pian-han-shu)
12. [new expression](https://en.cppreference.com/w/cpp/language/new)
13. [表达式](https://zh.cppreference.com/w/cpp/language/expressions)
14. [字面量](https://zhxilin.github.io/post/tech_stack/1_programming_language/modern_cpp/cpp11/user_defined_literal/#%E5%AD%97%E9%9D%A2%E9%87%8F)
15. [C++14 更多新特性](https://zhxilin.github.io/post/tech_stack/1_programming_language/modern_cpp/cpp14/more_cpp14/)
16. [std::initializer_list in C++ 1/2 - Internals and Use Cases](https://www.cppstories.com/2023/initializer_list_basics/#intro-to-the-stdinitializer_list)
