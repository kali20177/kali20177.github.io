# C++ 和 C 模块的相互调用


CMake 默认 `.c` 文件使用 gcc 编译，`.cpp` 文件使用 g++ 编译。**C++ 不兼容 C99 以后的特有语法**，比如指示器语法的乱序初始化、复合字面量等。

C/C++ 混合编程大致应该遵循：

1. C 语言和 C++ 的模块单独编译成库。
2. 接口部分（`.h/.hpp` 文件）不能出现特有语法。
3. 尽量不要在 C 代码中使用 C++ 关键字。

## 1. C++ 调用 C

C++ 调 C 要相对容易。以下面代码为例：

```c
// array_d.c
#include <stdio.h>
#include "array_d.h"

void printArray(int arr[], int size) {
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

struct MyStruct ss = {
    .y = 1,
    .x = 2,
};

void printMyStruct(struct MyStruct* s) {
    printf("%d, %d\n", s->x, s->y);
}

// array_d.h
# pragma once
struct MyStruct {
    int x;
    int y;
};

void printArray(int arr[], int size);
void printMyStruct(struct MyStruct* s);
```

可以看出，`array_d.c` 涉及到 C99 语法，而接口部分和 C++ 兼容，C++ 部分调用时使用 `extern "C"` 引入头文件：

```cpp
#include <iostream>
#include <iterator>
#include <vector>
#include <memory>

// 在 C 语言模块中定义的处理数组的函数
extern "C" {
    #include "array_d.h"
}

template <typename T>
std::unique_ptr<T[]> vectorToArray(const std::vector<T>& vec) {
    std::unique_ptr<T[]> arr(new T[vec.size()]);
    for (std::size_t i = 0; i < vec.size(); ++i) {
        arr[i] = vec[i];
    }
    return arr;
}

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    // 调用 vectorToArray 函数，将产生的数组作为参数传递给 C 风格的函数
    auto arr = vectorToArray(vec);
    printArray(arr.get(), vec.size());

    const int s = 6;
    int num[s] = {1, 2, 3, 4, 5, 6};
    printArray(num, s);
}
```

C++ 中允许使用常量表达式定义数组大小，可以看到这种方式的数组能正常调用 C 库函数。

由于 g++ 编译器编译过程中默认会在 .cpp 文件定义一个宏 `#define __cplusplus`，因此也可以这样：

```c
# pragma once
#ifdef __cplusplus 
extern "C" {
#endif  //__cplusplus

struct MyStruct {
    int x;
    int y;
};

void printArray(int arr[], int size);
void printMyStruct(struct MyStruct* s);

#ifdef __cplusplus
}
#endif  //__cplusplus
```

这种方式能避免在 cpp 文件调用 `array_d.h` 时显式使用 `extern "C"`。

另外，C++ 中可以继承来自 C 模块的结构体，例如对于：

```c
// usart.h
# pragma once
#include <stdint.h>
typedef struct  {
    uint32_t baud_rate;
    uint16_t stop_bits;
} USART_InitDef;
```

可以直接使用 public 继承，但是注意派生类的构造函数定义需要调用"基类"的默认构造函数。建议使用 `{ }` 调用 C 结构体的默认构造，使用 `( )` 在 clang-15 编译不通过。

```cpp
struct MyUsart : public USART_InitDef {
    using BaudRate_t = const uint32_t;
    using StopBits_t = const uint16_t;
public:
    MyUsart(BaudRate_t baud_rate, StopBits_t stop_bits) 
        : USART_Def{baud_rate, stop_bits} { }
    MyUsart() = delete;
    MyUsart(const MyUsart&) = delete;
    MyUsart(MyUsart&&) = delete;
};
```

在嵌入式场景应用中，可以用 C++ 继承 C 模块的结构体，使用**链式调用**配置设备：

```c
// device.h
#pragma once
typedef struct {
    int baudRate;
    int dataBits;
    int stopBits;
} DeviceConfig;
```

> 链式调用：每个成员方法都返回对象的引用，一种实现函数式编程的方式。注意成员函数之间应该互相解耦，达到调用顺序无关的效果。 

```cpp
// device.cpp
class Device : public DeviceConfig {
public:
    Device& setBaudRate(int rate) {
        baudRate = rate;
        return *this;
    }
    Device& setDataBits(int bits) {
        dataBits = bits;
        return *this;
    }
    Device& setStopBits(int bits) {
        stopBits = bits;
        return *this;
    }
    void printConfiguration() {
        std::cout << "Baud Rate: " << baudRate << std::endl;
        std::cout << "Data Bits: " << dataBits << std::endl;
        std::cout << "Stop Bits: " << stopBits << std::endl;
    }
};

// init
Device device;
device.setBaudRate(9600).setDataBits(8).setStopBits(1).printConfiguration();
```

## 2. C 调用 C++

对于普通函数，只需要接口兼容 C 语言语法并像上面一样使用 `__cplusplus` 宏包装。

对于类方法，可以给每个方法写“适配层”。例如下面的例子：

```cpp
// Person.h
#pragma once
#include <string>

class Person {
public:
    Person(const std::string& name, int age);
    ~Person();

    void sayHello();
    void setAge(int age);
    int getAge();

private:
    std::string name;
    int age;
};

// Person.cpp
#include "Person.h"
#include <iostream>

Person::Person(const std::string& name, int age) : name(name), age(age) {}
Person::~Person() {}

void Person::sayHello() {
    std::cout << "Hello, my name is " << name << ", I am " << age << " years old." << std::endl;
}

void Person::setAge(int age) {
    this->age = age;
}

int Person::getAge() {
    return age;
}
```

定义给 C 调用的 API：

```cpp
// PersonWrapper.h
#pragma once
#ifdef __cplusplus
extern "C" {
#endif

// C 风格的接口函数声明
typedef void* PersonPtr;

//------------------------------------------------------------
// 类方法
//------------------------------------------------------------
PersonPtr createPerson(const char* name, int age);
void deletePerson(PersonPtr person);
void personSayHello(PersonPtr person);
void personSetAge(PersonPtr person, int age);
int personGetAge(PersonPtr person);

//------------------------------------------------------------
// 普通函数
//------------------------------------------------------------
void testfunc(int argc);

#ifdef __cplusplus
}
#endif

// PersonWrapper.cpp
#include "PersonWrapper.h"
#include "Person.h"

PersonPtr createPerson(const char* name, int age) {
    return new Person(name, age);
}

void deletePerson(PersonPtr person) {
    delete static_cast<Person*>(person);
}

void personSayHello(PersonPtr person) {
    static_cast<Person*>(person)->sayHello();
}

void personSetAge(PersonPtr person, int age) {
    static_cast<Person*>(person)->setAge(age);
}

int personGetAge(PersonPtr person) {
    return static_cast<Person*>(person)->getAge();
}

void testfunc(int argc){
    std::cout << "testfunc : "<< argc << "\n";
    return;
}
```

调用：

```c
#include "Person_c_api.h"
#include <stdio.h>

int main(int argc, char *argv[]) 
{
    PersonPtr person = createPerson("John", 30);
    personSayHello(person);
    personSetAge(person, 31);
    printf("New age: %d\n", personGetAge(person));
    deletePerson(person);

    testfunc(3);
    return 0;
}
```

> 对于函数重载问题，需要写多个不同的函数分别处理。

也可以直接在 C++ 文件中显式使用 `extern "C"` 导出给 C 模块使用的接口，例如下面的例子：

```cpp
#include "event.hpp"
#include <string>
#include <iostream>

class Base {
private:
    std::string str;
public:
    Base(std::string s) 
        : str(s) { }
    void PrintBase() 
        std::cout << str << "\n";
};

void EventLoopCpp()
{
    std::cout << " -- C++ function here -- " << "\n";
}

extern "C"  {
    void EventLoopC()  {
        Base b(" -- class here --");
        b.PrintBase();
        EventLoopCpp();
    }
}
```

然后在对应的 `.h/.hpp` 文件中分别包含这两类接口，C 语言中直接包含 `.h/.hpp` 调用 `EventLoopC();` 即可。

> #include 只做展开，所以能够直接包含 .hpp 文件。

```cpp
#ifndef EVENTLOOP_HPP_
#define EVENTLOOP_HPP_

// C++ API
void EventLoopCpp();
 
// C API
#ifdef __cplusplus
extern "C"
{
#endif

    void EventLoopC();  

#ifdef __cplusplus
}
#endif

#endif /* EVENTLOOP_HPP_ */
```

## 参考

1. [C/C++ 中的 extern 和 extern "C" 关键字的理解和使用](https://blog.csdn.net/m0_46606290/article/details/119973574)
2. [分析解决 sorry, unimplemented: non-trivial designated initializers not supported](https://www.jianshu.com/p/4ff7a4fdf163)
3. [数组乱序初始化：sorry, unimplemented: non-trivial designated initializers not supported](https://www.cnblogs.com/tengzijian/p/14966759.html)
4. [在 GCC 环境中开发 C 和 C++ 混合项目](https://hjk.life/posts/gcc-c-cpp/)
5. [如何从 C 中调用 C++ 函数](https://zhuanlan.zhihu.com/p/361485807)
6. [C、C++ 混合调用](https://www.cnblogs.com/xuanyi-chen/p/c-cpp-call.html)
7. [C 语言编译错误：Variably modified array at file scope](https://www.cnblogs.com/lfri/p/11131871.html)
8. [how-to-use-cpp-with-stm32-hal](https://barenakedembedded.com/how-to-use-cpp-with-stm32-hal/)
