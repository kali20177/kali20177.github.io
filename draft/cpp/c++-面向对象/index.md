# [未完成] C++ 面向对象


## 类

### 访问权限 

private 成员：
- private 私有成员，只能在定义该类的成员函数内部访问。对于类的外部不可见，外部对象无法直接访问，**派生类也无法访问私有成员**。
- private 成员通常用于封装类的实现细节，隐藏内部实现，提高安全性和封装性。

protected 成员：
- protected 也是类的私有成员，具有更宽松的访问权限。可以被该类的成员函数和派生类的成员函数访问。外部对象无法直接访问。
- protected 成员通常用于实现**继承和派生类的访问控制**。

**可使用 using 改变 protected 成员在派生类中的访问权限**，例如：

```cpp
#include <iostream>

class Base {
public:
    Base() {}
protected:
    int m = 0;
};

class Derived1 : public Base {
public:
    int test() {
        return m;
    }
};

class Derived2 : public Base {
public:
    using Base::m;
};

int main() 
{
    Derived1 d1;
    Derived2 d2;
    int x = d1.test();  // 只能通过成员函数访问
    int y = d2.m;       // 修改成了 public
    return 0;
}
```

### 成员函数

#### 隐式内联

**类内定义的成员函数**隐式内联（因为一般这种定义会出现在头文件中，被多处包含），类外定义的成员函数非隐式内联。例如：

```cpp
class Entity {
    // implicitly inline
    void foo1() {    std::cout << "foo1" << std::endl;    }
};
void foo2() {
    std::cout << "foo2" << std::endl;
}
```

#### 成员函数的 const

const 修饰类的方法，表明该方法不会改变对象的状态，且能正确处理 const 对象的调用。

```cpp
class Entity {
private:
    int m_X;
public:
    int GetX() {  return m_X;  }
    int GetX() const {  return m_X;  }
    void SetX(int x) {  m_X = x;  }
};

void PrintEntity(const Entity& e) {
    std::cout << e.GetX() << std::endl;
}
```

当使用常量左值引用传递对象时，const 保证了 GetX() 方法的调用是安全的。

> const 成员函数可以被 const 和非 const 对象调用。所以如果类方法不会改变类的状态，尽量使用 const 修饰。

标准库中经常提供 const 和非 const 版本的重载，处理返回值在常量性方面的不同。

### static 成员变量

**静态成员变量被该类的所有对象共享**，是位于类的作用域区域内的全局变量，不与某个具体对象关联。可以通过类名和作用域解析运算符访问和初始化。

```cpp
struct X
{
    static int s_value;        // 静态成员变量声明
};
int X::s_value{ 10 };  // 静态成员变量定义
int main() {
    X s;
    std::cout << X::s_value << std::endl;
}
```

> ODR use : 
> 1. 一个对象在它的值被读取（除非它是编译时常量）或写入，或取它的地址，或者被引用绑定时，这个对象被 ODR 使用。
> 2. 使用“所引用的对象在编译期未知”的引用时，这个引用被 ODR 使用。
> 3. 一个函数在被调用或取它的地址时，被 ODR 使用。
>
> 如果一个对象、引用或函数被 ODR 使用，那么程序中必须有它的定义；否则通常会有链接时错误。

因此，在上面的例子中，如果没有在类外定义静态变量则编译时报错。

除此之外，静态成员变量可以直接在类内初始化：

```cpp
class X
{
public:
    static const int s_value1  { 4 };   // 这是 "声明"
    static inline int s_value2  { 5 };   // C++ 17, 定义
    static constexpr int s_value3  { 6 };   // C++ 17 , 内联变量定义，constexpr 成员隐式内联
};
const int X::s_value1;  // 需要此行
```

注意上方的 s_value1，这种方式下仍然属于声明，可以使用他的值，但取地址行为未定义（虽然部分编译器可能违反这条规则进而编译通过）。

### static 成员函数

由于静态成员函数不与对象关联，所以默认参数无 *this 指针。静态成员函数可直接访问其他静态成员（变量或函数），不能访问非静态成员。这是因为非静态成员必须属于一个此类的对象，而静态成员函数没有类对象可供使用。

静态成员函数可以在类外定义，定义时不需要 static 修饰：

```cpp
class Log
{
private:
    static inline std::string str = "Hello World";
public:
    static void setStr(std::string&& message);
};

void Log::setStr(std::string&& message) {
    str = message;
    std::cout << str << std::endl;
}
```

> C++ 没有静态构造函数

### 友元

friend 声明在类体内，授予一个函数或类完整的访问权限。

> 构造函数与析构函数也可以是友元，友元关系不能传递，不能继承。

#### 友元函数

某类的友元函数不是该类的成员函数

### 构造函数

构造函数没有名字且无法被直接调用。它们在发生初始化时被调用，且按照初始化的规则进行选择。

1. 成员初始化：
   
   > 对象实例化过程中，内存分配后，才会调用构造函数初始化对象的成员变量。

    1. 构造函数体内赋值
    
        直接将变量赋值过程写在构造函数体中，不建议使用。函数体内赋值相比于成员初始化列表存在效率问题，例如：

        ```cpp
        class Example
        {
        public:
            Example() {  std::cout << "create example " << "\n";  }
            Example(int x) {  std::cout << "create example with " << x << "\n";  }
        };

        class Entity 
        {
        private:
            std::string m_Name;
            Example m_Example;
        public:
            Entity() : m_Name("Unknown") {  m_Example = Example(8);  }  // 打印两次
            Entity() : m_Name("Unknown"), m_Example(8) { }              // 打印一次
        };

        Entity e;
        // Entity e();  // 警告
        // Entity e{};  // OK
        ```

        在进入构造函数的函数体之前，m_Example 进行了默认初始化。

        **在开始执行组成构造函数体的复合语句之前，所有直接基类、虚基类和非静态数据成员的初始化均已结束**。

        > 这里注意调用默认无参构造函数时，如果使用 `Example e();` 编译器会出现警告：`warning: empty parentheses interpreted as a function declaration [-Wvexing-parse]`。这种写法和函数声明的写法存在歧义，变量 e 不能被正常构造，要去掉 `()` 或者使用 `{}`。

    2. 成员初始化器列表（勿和 initializer_list 混淆）

        - **成员初始化器列表的构造顺序是按照类中成员变量声明的顺序**，而非成员初始化器列表本身的顺序。

            ```cpp
            struct X
            {
                int i, j;
                X(int val) : j(val), i(j)  { }  // 先初始 i, 再初始 j
            };
            ```

        - **引用成员不能绑定到成员初始化器列表中的临时量**：

            ```cpp
            struct A
            {
                A() : v(42) {} // 错误
                const int& v;
            };
            ```
        
            > 上面的代码在 gcc, msvc 下可以正常编译，clang 下才会提示：error: reference member 'v' binds to a temporary object whose lifetime would be shorter than the lifetime of the constructed object。这是由于在提案 CWG1696 前，构造函数的成员初始化器列表中可以绑定临时量到引用成员。但是这个临时量只持续到构造函数退出前，而非对象存在期间。

        - **如果成员是 const、引用或未提供默认构造的类类型**，必须使用成员初始化器列表。

            ```cpp
            class Base
            {
            public:
                Base(int a, int b) : m_a(a), m_b(b) // OK 
                {
                    // m_a = a;        // error
                    // m_b = b;        // error
                }
            private:
                int& m_a;
                const int m_b;
            };
            ```

    3. 委托构造函数

        允许在构造函数初始化列表中调用另一个构造函数，委托完成部分初始化，定义委托构造时没有顺序要求，例如：
            
        ```cpp
        class Person {
        private:
            std::string name;
            int age;
        public:
            Person() : Person("Unknown", 0) {}
            Person(const std::string& name) : Person(name, 0) {}
            // 委托构造函数
            Person(const std::string& name, int age) : name(name), age(age) {}
        };

        Person person2("Alice");
        ```

        前两个构造函数将都会委托给第三个构造函数完成实例化。

        使用委托构造时不能使用构造函数的初始化列表：

        ```cpp
        class class_a {
        public:
            class_a() {}
            // member initialization here, no delegate
            class_a(string str) : m_string{ str } {}

            //can't do member initialization here
            // error C3511: a call to a delegating constructor shall be the only member-initializer
            class_a(string str, double dbl) : class_a(str) , m_double{ dbl } {}

            // only member assignment
            class_a(string str, double dbl) : class_a(str) { m_double = dbl; }
            double m_double{ 1.0 };
            string m_string;
        };
        ```

        使用委托构造时要注意**防止形成环形构造**。

2. 默认构造函数

    可以不带任何实参调用的构造函数是默认构造函数。

3. constexpr 构造
    
    被 constexpr 修饰的构造函数会将该类型转为 LiteralType，令常量对象在编译期被构造，分配在 rodata 段。

### 转换构造函数

不以说明符 explicit 声明的构造函数被称为转换构造函数（converting constructor）。

### 析构函数

> 析构函数内定义对象被销毁前要执行的动作，例如回收堆内存，记录日志等。

**RAII**：资源获取即初始化（Resource Acquisition Is Initialization），将使用前请求的资源（堆内存、执行线程、打开的套接字、打开的文件、锁定的互斥量、磁盘空间、数据库连接等——任何受限的资源）的生命周期与一个对象的生存期绑定。

### 复制构造函数

可以接收同类型的另一对象为实参的构造函数是复制构造函数或移动构造函数。

> 浅拷贝：
> 深拷贝：

### 移动构造函数

### this 指针

`this` 指针始终指向正在被操作的对象。当对某个对象调用**非静态成员函数**时，编译器会将该对象的地址作为隐藏的参数传递给函数。

```cpp
myDate.setMonth( 3 );
// 等价于
setMonth( &myDate, 3 );
```

对象的 this 指针不是对象的一部分，他只是一个函数参数，所以不会影响 sizeof 语句的结果。

> All non-static member functions have a **this const pointer** that holds the address of the implicit object.

用途：

1. 显式调用 this 指针消除成员变量和同名参数的歧义
   
    ```cpp
    void PrintEntity(const Entity& e) {
        // print logic
    }
    class Entity {
    public:
        int x, y;
        Entity(int x, int y) {
            this->x;
            (*this).y = y;
            PrintEntity(*this);
        }
    };
    ```

2. 通过返回 this 指针实现链式调用
3. 重置类的状态

    ```cpp
    void Reset() {  *this = {};  }
    ```

> this 指针可以被 delete。

## 继承

继承时也存在 private, protected 和 public 三种继承，前两种很少用到。且默认是 private 继承。

```cpp
class A {};
class B : public A {};
class C : private B {};
class D : protected C {};
class E : D {};
```

### 虚函数

如果一个成员函数是虚函数，则派生类中匹配的重写都是虚函数。派生类中的虚函数重写不会隐式地让基类成员函数成为虚函数。

> 条款 9 : 不要从构造函数或析构函数调用虚函数。
> rule 9 : Never call virtual functions during construction or destruction.    —— Effective C++

> 条款 36 : 绝不重新定义继承而来的 non-virtual 函数
> rule 36 : Never redefine an inherited non-virtual function.    —— Effective C++

> 条款 37 : 绝不重新定义继承而来的缺省参数值 (virtual 函数)
> rule 37 : Never redefine a function's inherited default parameter value.    —— Effective C++

创建派生类时首先调用基类构造函数，此时派生类还没有创建，无法调用派生类的虚函数版本。在基类析构函数中调用虚函数，它将始终解析为该函数的基类版本，因为派生部分已经被销毁。

### override and final

派生类虚函数只有当它的函数签名和返回类型完全匹配时才被认为是重写。

> C++ Templates -- the complete guides
> 
> 签名：
> 1. The unqualified name of the function
> 2. The class or namespace scope of that name, and if the name has internal linkage, the translation unit in which the name is declared
> 3. The const, volatile, or const volatile qualification of the function
> 4. The types of the function parameters
> 5. its return type, if the function is generated from a function template
> 6. The template parameters and the template arguments, if the function is generated from a function template
>
> 如果函数具有不同的签名，则它们可以在程序中共存。普通函数的签名不包含返回类型。

因此在重写派生类的虚函数时可能会导致意外结果，例如下面两种情况会编译通过：

```cpp
class A {
public:
	virtual std::string_view getName1(int x) { return "A"; }
	virtual std::string_view getName2(int x) { return "A"; }
};
class B : public A  {
public:
	virtual std::string_view getName1(short x) { return "B"; }     // note: parameter is a short
	virtual std::string_view getName2(int x) const { return "B"; } // note: function is const
};
```

override 关键字用于虚函数后，明确表示此处重写基类中的虚函数，不匹配则编译失败：

```cpp
class A {
public:
	virtual std::string_view getName1(int x) { return "A"; }
	virtual std::string_view getName2(int x) { return "A"; }
	virtual std::string_view getName3(int x) { return "A"; }
};
class B : public A  {
public:
	std::string_view getName1(short int x) override { return "B"; } // compile error, function is not an override
	std::string_view getName2(int x) const override { return "B"; } // compile error, function is not an override
	std::string_view getName3(int x) override final { return "B"; } // okay, function is an override of A::getName3(int)
};
```

> 对于派生类的覆写函数使用 override 说明符，包括虚析构。

final 说明符用于表示此虚函数不再允许被覆写。

### 纯虚函数

```cpp
virtual std::string speak() const = 0; 
```

任何具有一个或多个纯虚函数的类都会变成抽象基类，不能被实例化。它要求派生类必须重写此成员函数，否则此派生类也会被认为是抽象类。

> 1. 抽象类具有虚表
> 2. 任何具有纯虚函数的类都应该具有虚析构函数
> 3. 没有成员变量，全部有虚函数组成的类被称为接口类，对应其他语言的接口类 interface 的概念

> 可以为纯虚函数在类外提供定义。-- C++ Primer

如果要为抽象类提供纯虚函数的默认实现（很少用到），在类外定义虚函数的实现，并在派生类中显式通过作用域运算符调用：

```cpp
class Animal
{
protected:
    std::string m_name {};
public:
    Animal(std::string_view name)
        : m_name(name)	{	}
    const std::string& getName() const { return m_name; }
    virtual std::string speak() const = 0; 
    virtual ~Animal() = default;
};

// 定义默认版本
std::string Animal::speak() const	{
    return "buzz"; 
}

class Dragonfly: public Animal
{
public:
    Dragonfly(std::string_view name)
        : Animal{name}	{	}

    std::string speak() const override	{
        return Animal::speak();  // 使用默认实现
    }
};
```

作用域解析运算符同时可用于区分继承类中的同名函数：

```cpp
class Base
{
private:
	int m_value {};
public:
	Base(int value)
		: m_value{ value }	{	}
    void identify() const { std::cout << "Base\n"; }
};

class Derived: public Base
{
public:
    Derived(int value)
        : Base { value }	{	}
    void identify() const	{
        Base::identify(); // call Base::identify()
        std::cout << "Derived\n"; // then identify ourselves
    }
};
```

### 虚析构

在析构函数未使用 `virtual` 修饰的情况下，使用基类类型的指针创建一个派生类对象，那么派生类析构时只会调用基类的析构函数。

> 条款 7 : 为多态基类声明 virtual 析构函数
> rule 7 : Declare destructors virtual in polymorphic base classes.    —— Effective C++

```cpp
...
class Base {
public:
    virtual ~Base() {
        std::cout << "Base destructor called" << std::endl;
    }
};

class Derived : public Base {
public:
    ~Derived() {
        std::cout << "Derived destructor called" << std::endl;
    }
};

int main() {
    Base* ptr = new Derived();
    delete ptr;  // 如果 Base 的析构函数不是虚函数，只会调用 Base 的析构函数
    return 0;
}
```

### 多重继承

> 尽量避免使用多重继承，因为要处理菱形继承问题。

假设 B、C 都继承自 A，D 继承自 B 和 C，不使用虚继承的情况下 D 会得到 A 类的两个副本。虚继承用于解决此类问题。

```cpp
class A {   };
class B: virtual public A    {   };
class C: virtual public A    {   };
class D: public B, public C  {   };
```

> 此时，"最派生"的类负责构造虚基类 A

## 多态

通过基类指针操作对象，可以降低重复，提高代码可维护性。还可以将接口和实现分离，实现解耦。

### 绑定的概念

**绑定**指将标识符 (如变量和函数名) 转换为地址的过程。分为静态绑定和动态绑定。
- Early Binding (compile-time time polymorphism) As the name indicates, compiler (or linker) directly associate an address to the function call.
- Late Binding : (Run time polymorphism) In this, the compiler adds code that identifies the kind of object at runtime then matches the call with the right function definition. This can be achieved by declaring a virtual function.

### RTTI 机制

Run-Time Type Identification, 用于在程序运行时获取对象的类型信息，主要包括 typeid 和 dynamic_cast 运算符。

> 注意：某些 dynamic_cast 需要 RTTI，但是另一些实现选择在编译时完成，不依赖 RTTI。参考 cppreference 和 [Itanium C++ ABI](http://itanium-cxx-abi.github.io/cxx-abi/abi.html#dynamic_cast)

> typeid 和 decltype：
> - typeid 在运行时获取类型信息，返回 type_info 对象，用于动态类型识别。
> - decltype 在编译时获取类型信息，用于类型推断、声明变量类型、模板编程等。decltype 读作 "declare type"。

#### typeid

使用 typeid 运行时获取类型信息，但是返回的结果是实现定义的。其中 MSVC、IBM、Oracle 编译期会生成人类可读的类型名，gcc 与 clang 返回重整名（mangled name），这是由于二者采用 Itanium C++ ABI。

> Itanium C++ ABI:
> 最早是为 Intel 的安腾（Itanium）架构编写的，是一个跨架构的 C++ ABI。上世纪 90 年代，GCC 编译器通过使用 `setjmp` 和 `longjmp` 实现了 C++ 异常处理。出于多种原因，这是不能令人满意的。Intel 生产 Itanium 时，他们希望允许 GCC 和英特尔编译器 icc 进行互操作，包括在两个编译器编译的代码之间抛出异常。他们成立了一个工作组来设计一个 ABI 来支持这一点，其中包括一种不使用 longjmp 的异常处理方法。
> MSVC 不使用 Itanium C++ ABI

重整名可以用实现指定的 API 转换到人类可读的形式，例如：

```cpp
#include <typeinfo>  // 提供 typeid
#include <cxxabi.h>
#include <memory>
#include <cstdlib>

std::string demangle(const char* name) {
    int status = -1; 
    std::unique_ptr<char, void(*)(void*)> res {
        abi::__cxa_demangle(name, NULL, NULL, &status),
        std::free
    };
    return (status==0) ? res.get() : name ;
}

// 使用
std::cout << demangle(typeid(obj).name()) << std::endl;
```

也可以使用 `c++filt -t` 命令还原类型名：

```bash
> c++filt -t 4BaseI8Derived1E
Base<Derived1>
```

> c++fit 用于还原由于名字重整（name mangling）机制的名称。例如使用 nm 列出二进制文件的符号表后，还原符号 _ZN4BaseI8Derived2E9interfaceEv：
> ```bash
> c++filt _ZN4BaseI8Derived2E9interfaceEv                    
> Base<Derived2>::interface()
> ```

#### dynamic_cast

沿继承层级向上、向下及侧向，安全地转换到其他类的指针和引用。主要有下面三种用法：

```cpp
// type 必须是类类型
dynamic_cast<type*>(e)   // e 必须是指针 
dynamic_cast<type&>(e)   // e 必须是左值
dynamic_cast<type&&>(e)  // e 非左值
```

如果转换失败且目标类型是指针，则返回该类型的空指针。如果转换失败且目标类型是引用类型，则会抛出与类型 std::bad_cast 的处理块匹配的异常。

static_cast 也能用来进行向下转换，不会有运行时检查的开销，但只有在程序（通过其他逻辑）能够保证表达式指向的对象肯定是 Derived 时才是安全的。

### 对象切片

当使用基类类型，利用拷贝初始化的方式实例化派生类对象时，会发生切片现象。基类无法容纳派生类的所有成员。以下面代码为例：

```cpp
class Base {
private:
    int a, b;
public:
    Base(int _a, int _b) : a(_a), b(_b) {}
    virtual void foo() const {}
};

class Dev1 : public Base {
private:
    bool c;
public:
    Dev1(int _a, int _b, bool _c) : Base(_a, _b), c(_c) {}
    void foo() const {
        std::cout << "Dev1 foo" << "\n";
    }
};

class Dev2 : public Base {
private:
    bool c;
public:
    Dev2(int _a, int _b, bool _c) : Base(_a, _b), c(_c) {}
    void foo() const {
        std::cout << "Dev2 foo" << "\n";
    }
};

void fooPrint(const Base& b) {
    b.foo();
}

int main() {
    Base b1 = Dev1(1, 2, true);         // 发生切片
    const Base& b2 = Dev1(2, 3, true);  // 给派生类创建基类引用，对象未被切片，但能通过引用使用的变量和函数不全
    b2.foo();

    // 多态实现
    const Base& d1 = Dev1(3, 5, true);
    const Base& d2 = Dev2(9, 4, false);
    fooPrint(d1);
    fooPrint(d2);
    return 0;
}
```

可以使用基类的引用，在不使用 new 的情况下实例化派生类对象，并且能够实现多态性。

## 运算符重载

### 限制
1. 不能重载 `::`（作用域解析）、`.`（成员访问）、`.*`（通过成员指针的成员访问 `a.*b`）及 `?:`（三目条件）运算符。
2. 不能创建新运算符，例如 `**`、`<>` 或 `&|`。
3. 无法改变运算符的优先级、结合方向或操作数的数量。
4. 重载的运算符 `->` 必须要么返回裸指针，要么（按引用或值）返回同样重载了运算符 `->` 的对象。
5. 运算符 `&&` 与 `||` 的重载会失去短路求值。

> `&&`、`||` 和 `,`（逗号）在被重载时失去它们特殊的定序性质，并且即使不使用函数调用记法，也表现为与常规的函数调用相似。

一些资料中也提到了不能重载 `sizeof`。 

> **不是所有运算符都可以重载为友元函数**
> 赋值 `=` 、下标 `[]` 、函数调用 `()` 和成员选择 `->`运算符必须重载为成员函数 (历史原因)。
> **不是所有运算符都可以重载为成员函数**
> 通常，如果左操作数不是类（例如 int），或者是不能修改的类（例如 std：：ostream），无法使用成员重载。

经验法则：

1. 赋值 `=` 、下标 `[]` 、函数调用 `()` 或成员选择 `->`，作为成员函数重载。
2. 一元运算符，作为成员函数重载。
3. 对不修改其左操作数的二元运算符（例如 operator+），作为一个普通函数（首选）或友元函数重载。
4. 对修改左操作数的二元操作符，但你不能向左操作数的类定义添加成员（例如 operator<<，左操作数 ostream），作为普通函数（首选）或友元函数重载。
5. 对修改左操作数的二元运算符（例如 operator+=），并且可以修改左操作数的定义，作为一个成员函数重载。

### opeartor=

> 条款 10 : 令 operator= 返回一个 reference to *this  
> rule 10 : Have assignment operators return a reference to *this.    —— Effective C++

> 条款 11 : 在 operator= 中处理“自我赋值”
> rule 11 : Handle assignment to self in operator=.    —— Effective C++

## 相关设计模式

### 组合

### 委托

一个对象委托另一个对象执行特定的任务或操作。一般通过包含委托类指针或者引用实现。

```cpp
#include <iostream>

// 委托类
class Delegate {
public:
    void doTask() {
        std::cout << "Delegate is performing the task." << std::endl;
    }
};

// 委托对象
class Delegator {
private:
    Delegate* delegate;

public:
    Delegator(Delegate* del) : delegate(del) {}

    void assignTask() {
        if (delegate) {
            delegate->doTask(); // 通过指针委托给 Delegate 对象执行任务
        } else {
            std::cout << "No delegate assigned." << std::endl;
        }
    }
};

int main() {
    Delegate delegate;
    Delegator delegator(&delegate);
    delegator.assignTask();

    // 修改委托对象
    Delegate anotherDelegate;
    delegator = Delegator(&anotherDelegate);
    delegator.assignTask();

    return 0;
}
```

### pimpl

### 奇异递归模板模式 CRTP

CRTP 可用于在基类暴露接口而派生类实现该接口时实现“编译期多态”。

```cpp
// 基类模板
template <typename Derived>
class Base {
public:
    void interface() {
        // 下转型
        static_cast<Derived*>(this)->impl();
    }
    void impl() {
        std::cout << "Base" << std::endl;
    }
};
// 派生类 1
class Derived1 : public Base<Derived1> {
public:
    void impl() {
        std::cout << "Derived1" << std::endl;
    }
};
// 派生类 2
class Derived2 : public Base<Derived2> {
public:
    void impl() {
        std::cout << "Derived2" << std::endl;
    }
};
// 多态 func
template<typename T>
void func(Base<T>& base) {
    base.interface();
}
int main(int argc, char* argv<::>) {
    Derived1 d1;
    Derived2 d2;
    func(d1);
    func(d2);
    return 0;
}
```

注意，这种方式需要在基类中向派生类添加功能，被称为颠倒继承（Upside Down Inheritance），和传统的实现多态方式相反。

CRTP 能避免虚函数开销，实现编译时的静态多态性。但是编译后的二进制文件可能变大。采用 hands on design patterns with c++ 第八章代码进行性能测试，参见[测压结果](https://quick-bench.com/q/vORXUiFA_978rVueP-BuZH-AwwM)，**CRTP 的性能需要打开优化后体现**。

## 参考

1. [Member functions](https://www.learncpp.com/cpp-tutorial/member-functions/)
2. [Const class objects and const member functions](https://www.learncpp.com/cpp-tutorial/const-class-objects-and-const-member-functions/)
3. [定义与 ODR（单一定义规则）](https://zh.cppreference.com/w/cpp/language/definition)
4. [友元声明](https://zh.cppreference.com/w/cpp/language/friend)
5. [引用初始化](https://zh.cppreference.com/w/cpp/language/reference_initialization)
6. [编译期对象构造优化 .bss 为 .rodata](https://microcai.org/2024/01/22/constexpr-constructor.html)
7. [转换构造函数](https://zh.cppreference.com/w/cpp/language/converting_constructor)
8. [委托构造函数](https://learn.microsoft.com/zh-cn/cpp/cpp/delegating-constructors?view=msvc-170)
9. [C++11 新语法之委托构造函数](https://blog.csdn.net/KingOfMyHeart/article/details/107732764)
10. [深入探索 C++ 多态 - 虚析构](https://zhuanlan.zhihu.com/p/654786933)
11. [成员列表的作用](https://www.bilibili.com/video/BV1Zi4y1F7o1/?spm_id_from=333.788&vd_source=d669a99eaef48bca1935ad7a52416701)
11. [C++ 面向对象拓展](https://ustccoder.github.io/2020/07/28/C++_oop_4/)
12. [`this` 指针](https://learn.microsoft.com/zh-cn/cpp/cpp/this-pointer?view=msvc-170)
13. [15.1 — The hidden “this” pointer and member function chaining](https://www.learncpp.com/cpp-tutorial/the-hidden-this-pointer-and-member-function-chaining/)
14. [Virtual functions and polymorphism](https://www.learncpp.com/cpp-tutorial/virtual-functions/)
15. [Is the return type part of the function signature?](https://stackoverflow.com/questions/290038/is-the-return-type-part-of-the-function-signature)
16. [第 18 篇：C++ 静态绑定和动态绑定](https://zhuanlan.zhihu.com/p/192178632)
17. [object slicing](https://www.learncpp.com/cpp-tutorial/object-slicing/)
18. [Pure virtual functions, abstract base classes, and interface classes](https://www.learncpp.com/cpp-tutorial/pure-virtual-functions-abstract-base-classes-and-interface-classes/)
19. [C++ RTTI](https://www.zhihu.com/question/524268001/answer/2410267306)
20. [dynamic_cast 转换](https://zh.cppreference.com/w/cpp/language/dynamic_cast)
21. [Itanium C++ ABI new](https://news.ycombinator.com/item?id=30399707)
22. [运算符重载](https://zh.cppreference.com/w/cpp/language/operators)
23. [Overloading operators using member functions](https://www.learncpp.com/cpp-tutorial/overloading-operators-using-member-functions/)
24. [Design Patterns With C++（八）CRTP（上）](https://zhuanlan.zhihu.com/p/142407249)
