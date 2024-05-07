# C99 语法


## 1. 指示器语法 designed initialiazer

数组初始化时的指示器语法：

```c
int a[10] = {0, 0, 29, 0, 0, 0, 0, 48, 0, 0};

// 等价于
int a[10] = {[2] = 29, [7] = 48};
```

还可以：

```c
int a[10] = { [1] = 1, [8 ... 9] = 10 };
```

结构体初始化：

```c
typedef struct {
    int number;
    char name[100];
    int on_hand;
} Part;

struct point { 
    int x;
    int y; 
};

// 初始化
Part part1 = {.on_hand = 10;
              .name = "Disk Drive",
              .number = 528};

struct point p1 = { .y = 2, .x = 1 };

// gcc old-style 可以使用不加 . 并结合 : 的初始化方式，在 clang 下会产生警告
struct point p2 = { y : 2, x : 1 }; 
```

[运行](https://godbolt.org/z/M146E81jP)

## 2. 复合字面量 compound literals

就地构造一个指定类型的无名对象，在只需要定义一次数组、结构体或联合体变量时使用。

```c
int b[] = {3, 0, 3, 4, 1};
total = sum_array(b, 5);

// 等价于
total = sum_array((int []){3, 0, 3, 4, 1});
```

```c
typedef struct {
    int number;
    char name[100];
    int on_hand;
} Part;

// 作为参数传入 
void print_part(struct part p) {
    printf("Part number : %d\n", p.number);
    printf("Part name : %s\n", p.name);
    printf("On hand : %d\n", p.on_hand);
}

// 传统方式
Part part1 = {528, "Disk drive", 10};

// 复合字面量
print_part((struct part){528, "Disk drive", 10})；
// 复合字面量 + 指示器
print_part((struct part){.on_hand = 10;
                         .name = "Disk Drive",
                         .number = 528});
```

{{< admonition type=note title="C++ 临时对象" open=true >}}
这个特性看起来有点像 C++ 的临时匿名对象，但是**复合字面量是左值**。
对于 C++ 临时对象，一般的处理方法是“小数据”(尺寸在两三个指针以内) 值传递，大数据传引用。 —— 《C++ 之旅》
{{< /admonition >}}

下面是 C++ 传递临时对象的方式：

```cpp
// 值传递
void myFunction1(std::vector<int> vec)
// 常量左值引用传递
void myFunction2(const std::vector<int> &vec)
// 右值引用传递
void myFunction3(std::vector<int> &&vec)
...
// 调用
myFunctionx({10, 20，30});
```

> 1. 一般来说，error：`cannot bind non-const lvalue reference of type '...' to an rvalue of type '...'` 是引用未加 const 导致的。
> 2. 常量左值引用和右值引用都可以用来绑定到临时对象（右值），右值引用主要用于实现移动语义和完美转发，可以避免不必要的对象复制。如果函数需要修改参数，或者要接受可能会变的临时对象，应该使用 `T&&`。

## 3. inline

## 4. flexible array member

柔性数组：在结构体中的最后一个元素允许是未知大小的数组。

柔性数组在 C 语言中有一些实际的应用场景，特别是在需要动态内存管理的情况下。

1. 动态字符串结构体：包含一个固定大小的缓冲区和一个柔性数组用于存储字符串数据。这种结构体可以方便地进行字符串的动态管理，比如在解析文本数据时，可以动态地分配和释放字符串内存。

    > Redis 中使用的字符串类型 sds，最后的 buf 使用了柔性数组。C 字符串只支持 ASCII 字符，并且中间不能被 '\0' 截断。sds 类型可用于实现二进制安全的字符串。

    ```c
    struct __attribute__ ((__packed__)) sdshdr8 {
        uint8_t len; /* used */
        uint8_t alloc; /* excluding the header and null terminator */
        unsigned char flags; /* 3 lsb of type, 5 unused bits */
        char buf[];
    };
    ```

2. 动态数组结构体：定义动态数组结构体，其中包含一个固定大小的部分和一个柔性数组用于存储数组元素。这种结构体可以用于实现动态大小的数组，比如在需要动态管理数组大小的情况下。

    ```c
    struct DynamicArray {
        size_t size;
        int data[]; // 柔性数组用于存储数组元素
    };
    ```

3. 动态数据结构：用于定义动态数据结构，比如树、图等数据结构中的节点结构体，其中柔性数组可以用于存储节点的子节点或邻接节点等动态数量的数据。

## 5. Pragma 运算符

`#pragma once` 是受到绝大多数现代编译器支持的非标准语用。但是这种方式具有局限性：

> Unlike header guards, this pragma makes it impossible to erroneously use the same macro name in more than one file. On the other hand, since with #pragma once files are excluded based on their filesystem-level identity, **this can't protect against including a header twice if it exists in more than one location in a project.**

标准做法是：

```c
#ifndef LIBRARY_FILENAME_H
#define LIBRARY_FILENAME_H
// 头文件的内容
#endif /* LIBRARY_FILENAME_H */
```

不仅可以保证同一个文件不会被包含多次，也能保证内容完全相同的两个文件（或者代码片段）不会被不小心同时包含。Google C++ Style Guide 中也是采用标准做法。

> 不要在使用包含防护时以 __ 开头，虽然互联网上大量代码都这样做，具体参考[What does double underscore ( __const) mean in C?](https://stackoverflow.com/questions/1449181/what-does-double-underscore-const-mean-in-c/1449301#1449301)

`#pragma pack` 控制后继定义的结构体和联合体的最大对齐。

## 6. 变长数组（VLA）

> It generates much more code, and much slower code (and more fragile code), than just using a fixed key size would have done ~ Linus Torvalds

变长数组缩写为 VLA(variable-length array)，它是一种在运行时才确定长度的数组，通常在栈上分配内存空间。C11 中这个特性变成了可选项。

```c
int n;
int array[n];
```

限制：

- n 和 array 必须位于同一个文件作用域
- 不可以用于 typedef
- 不可使用在结构体中
- 不可以申明为 static 变量
- 不可以申明为 extern 变量或 extern 变量的指针

{{< admonition type=warning title="限制" open=true >}}
misra 规则和 Google C++ Style Guide 都禁用 VLA。实际使用中确实会出现很诡异的 bug。
{{< /admonition >}}

## 7. 堆分配

建议使用 `calloc` 代替 `malloc` + `memset`.

## 8. sizeof 

语法：

```c
sizeof( 类型 )	(1)	
sizeof 表达式	(2)	
```

**除非用于 VLA，sizeof 是不求值表达式**且 sizeof 运算符能在整数常量表达式中使用。

若表达式的类型为 VLA，则求值表达式，在**运行时**计算其所求值的数组大小。

## 参考

1. [compound literals](https://zh.cppreference.com/w/c/language/compound_literal)
2. [C99 标准的新特性](https://www.cnblogs.com/wuyudong/p/c99-new-feature.html)
3. [浅谈 C++ 中的临时对象](https://blog.csdn.net/simon_2011/article/details/78188413)
4. [References, simply](https://herbsutter.com/2020/02/23/references-simply/)
5. [深入浅出 C 语言中的柔性数组](https://www.cnblogs.com/jinxiang1224/p/8468206.html)
6. [C 语言柔性数组](https://www.cnblogs.com/wuyudong/p/c-flexible-array.html)
7. [Redis 使用及源码剖析 - 2.Redis 简单动态字符串 (SDS)-2021-1-16](https://cloud.tencent.com/developer/article/1945286)
8. [现代化 C 使用体验](https://liujiacai.net/blog/2022/04/30/modern-c/)
9. [C 语言的 2016](https://www.infoq.cn/article/c-language-2016/)
10. [实现定义行为的控制](https://zh.cppreference.com/w/c/preprocessor/impl)
11. [C 语言中 #pragma once 的作用是什么？](https://juejin.cn/post/7090412395964661791)
12. [技术摘要 | 现代 C99, C11 标准下的 C 语言编程](https://www.elliot98.top/post/tech/modern_c_standard/)
13. [VLA removal](https://lkml.org/lkml/2018/3/7/621)
14. [C 语言中变长数组的陷阱](https://blog.csdn.net/hustang/article/details/120024411)


