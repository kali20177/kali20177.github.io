# 解毒 Setjmp & Longjmp


## 1. 简单案例

下面是用 setjmp 和 longjmp 实现的简单捕获除 0 异常的一段 C 代码：

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <setjmp.h>

jmp_buf exception_env;

void handle_divide_by_zero(int count) {
    printf("Divide by zero exception at iteration %d\n", count);
}

int divide(int a, int b, int count) {
    if (b == 0) {
        longjmp(exception_env, 1);
    }
    return a / b;
}

int main() {
    int result, count = 1;
    srand((unsigned int)time(NULL)); 
    while (1) {
        int a = rand() % 100; // 随机 a
        int b = rand() % 10;  // 随机 b

        if (setjmp(exception_env) == 0) {
            result = divide(a, b, count);
            printf("Iteration %d: Result of %d / %d = %d\n", count, a, b, result);
        } else {
            handle_divide_by_zero(count);
        }
        count++;
    }
    return 0;
}
```

直接从字面上分析，当每次运行除法函数时会判断是否除 0，如果是则执行 longjmp，longjmp 含义是“长跳转”，或者叫非本地跳转，第二个参数是发生跳转时携带的返回值。main 函数中如果返回值是 0 表示正常，否则执行异常 handle。

`setjmp` 函数的返回值有两种情况：一种是当 `setjmp` 第一次被调用时，它返回 0；另一种是当 `longjmp` 跳转回 `setjmp` 时，它返回 `longjmp` 的第二个参数值。

> **Return value**
>
> 1. ​0​ if the macro was called by the original code and the execution context was saved to env.
>
> 2. Non-zero value if a non-local jump was just performed. The return value is the same as passed to longjmp.

### 修改 1

以上这个例子虽然能正常工作，但是`setjmp(exception_env)` 出现在条件表达式时无法拿到返回值。如果要正常获取 longjmp 携带的 1，则需要改写成下面的形式：

```c
while (1) {
    int a = rand() % 100; // 随机 a
    int b = rand() % 10;  // 随机 b

    int res = setjmp(exception_env);

    if (res == 0) {
        result = divide(a, b, count);
        printf("Iteration %d: Result of %d / %d = %d\n", count, a, b, result);
    } else {
        handle_divide_by_zero(count);
    }
    count++;
}
```

修改后的形式更适合根据 longjmp 返回值分类讨论。**但是，这样的用法不符合标准。**

### 修改 2

虽然笔者在 Linux 和 Windows 环境下，setjmp 返回值直接赋值目前未使用出异常问题，网络上也有不少类似的例子，但在《深入理解计算机系统》第三版和《Linux/UNIX 系统编程手册》中提到，setjmp 的返回值直接赋给变量是非标准行为。

```c
rc = setjmp(env);    /* Wrong!  */
```

SUSv3 和 C99 规定，对 setjmp() 的调用只能在如下语境中使用。
  - 构成选择或迭代语句中（if、switch、while、do-while、for）的整个控制表达式。 `switch(setjmp(env))`
  - 作为一元操作符!（not）的操作对象，其最终表达式构成了选择或迭代语句的整个控制表达式。 `while(!setjmp(env))`
  - 作为比较操作（==、!=、<等）的一部分，另一操作对象必须是一个整数常量表达式，且其最终表达式构成选择或迭代语句的整个控制表达式。 `if(setjmp(env) > 10)` 
  - 作为独立的函数调用，且没有嵌入到更大的表达式之中。 `setjmp(env);`

其他行为属于未定义行为，并且 setjmp 的实际行为和优化选项也有关。在实际使用中，如果我们需要通过 longjmp 的返回值选择执行不同的异常处理过程，建议选择全局变量或者 switch-case 语句实现。

```c
#include ...

jmp_buf exception_env;
int exception_type = 0; // 全局变量，用于记录异常类型

int divide(int a, int b, int count) {
    if (b == 0) {
        exception_type = 1; // 设置异常类型为 1
        longjmp(exception_env, 1);
    }
    if (b == 1) {
        exception_type = 2; // 设置异常类型为 2
        longjmp(exception_env, 2);
    }
    return a / b;
}

void handle_divide_by_zero(int count) {
    printf("Divide by zero exception at iteration %d\n", count);
}

void handle_divide_by_one(int count) {
    printf("Divide by one exception at iteration %d\n", count);
}

int main(int argc, char *argv[]) {
    int result, count = 1;
    srand((unsigned int)time(NULL)); // 初始化随机数种子

    for (int i = 0; i < 100; i++) {
        int a = rand() % 100; // 产生随机数 a
        int b = rand() % 10;  // 产生随机数 b

        if (setjmp(exception_env) == 0) {
            // 正常执行
            result = divide(a, b, count);
            printf("Iteration %d: Result of %d / %d = %d\n", count, a, b, result);
        } else {
            // 异常处理
            if (exception_type == 1) {
                handle_divide_by_zero(count);
            } else if (exception_type == 2) {
                handle_divide_by_one(count);
            }
            // 重置异常类型
            exception_type = 0;
        }
        count++;
    }
    exit(0);
}
```

或者：

```c
int main(int argc, char *argv[]) {
    ...

    for (int i = 0; i < 100; i++) {
        int a = rand() % 100; // 产生随机数 a
        int b = rand() % 10;  // 产生随机数 b

        switch (setjmp(exception_env)) {
            case 0:
                // 正常执行
                result = divide(a, b, count);
                printf("Iteration %d: Result of %d / %d = %d\n", count, a, b, result);
                break;
            case 1:
                // 处理除以 0 的异常
                handle_divide_by_zero(count);
                break;
            case 2:
                // 处理除以 1 的异常
                handle_divide_by_one(count);
                break;
            default:
                // 处理其他异常
                break;
        }
        count++;
    }
    ...
}
```

## 2. 用途

### 2.1 异常机制

### 2.2 C 协程

### 2.3 测试

大多数 C 语言的测试框架都借助了 setjmp 和 longjmp 实现断言失败后跳出当前测试。

## 3. 局限性

这类功能使用需要非常小心，Misra Rule 21.4 直接禁用此功能。

## 4. C++

C++ 中也支持使用 setjmp 和 longjmp，在头文件 `<csetjmp>` 中定义。保存环境时需要使用 `std::jmp_buf`，使用 `std::longjmp` 跳转。支持和 C 语言互相跳转。

## 参考

1. [setjmp](https://zh.cppreference.com/w/c/program/setjmp)
2. [std::longjmp](https://zh.cppreference.com/w/cpp/utility/program/longjmp)
3. [Linux/UNIX 系统编程手册](https://book.douban.com/subject/25809330/)
4. [C 语言中 setjmp 和 longjmp](https://www.cnblogs.com/hazir/p/c_setjmp_longjmp.html)

