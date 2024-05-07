# C/C++ 代码质量


![fun](/code_quality/fun.png)

## 1. 复杂度

### 1.1 定义

圈复杂度 (Cyclomatic complexity，CC) 是一种衡量代码复杂度的标准。圈复杂度可以用来衡量一个模块判定结构的复杂程度，其数量上表现为独立路径的条数，也可理解为覆盖所有的可能情况最少使用的测试用例个数。**可作为代码重构的指导指标，也可以据此确定测试顺序**。

某些大厂（例如阿里）的新代码提交后，平台会计算每个方法的圈复杂度，提醒对复杂方法进行重构。

### 1.2 计算方式

程序语句可以转化为如下的控制流图。

![flow chat](/code_quality/flow_chat.png)

然后可以根据下面的公式计算圈复杂度，e 表示控制流图中边的数量，n 表示控制流图中节点的数量。

$$
V = e - n + 2
$$

实际开发过程中一般借助工具直接查看结果。

### 1.3 工具

对于 C 代码，检查工具推荐使用 [lizard](https://github.com/terryyin/lizard)。用 pip 安装后命令行直接 `lizard <file>` 即可。

| **圈复杂度** | **代码状况** | **可测性** | **维护成本** |
| :----------: | :----------: | :--------: | :----------: |
|     1~10     | 清晰、结构化 |     高     |      低      |
|    10~20     |     复杂     |     中     |      中      |
|    20-30     |   非常复杂   |     低     |      高      |
|     >30      |    不可读    |   不可测   |    非常高    |

输出结果解释：

```shell
================================================
  NLOC    CCN   token  PARAM  length  location  
------------------------------------------------
       9      3     79      1       9 print_logpage_life@31-39@./poweroff.cpp
       5      1     34      3       5 RecordData_Sys::RecordData_Sys@60-64@./poweroff.cpp
       3      1     14      0       3 RecordData_Sys::~RecordData_Sys@66-68@./poweroff.cpp
      11      4     79      0      11 RecordData_Sys::print_idealcontainer@70-80@./poweroff.cpp
       7      2     55      1       7 RecordData_Sys::push_elem_idealcontainer@82-88@./poweroff.cpp
      19      1    166      0      23 main@90-112@./poweroff.cpp
1 file analyzed.
==============================================================
NLOC    Avg.NLOC  AvgCCN  Avg.token  function_cnt    file
--------------------------------------------------------------
     80       9.0     2.0       71.2         6     ./poweroff.cpp

===============================================================================================================
No thresholds exceeded (cyclomatic_complexity > 15 or length > 1000 or nloc > 1000000 or parameter_count > 100)
==========================================================================================
Total nloc   Avg.NLOC  AvgCCN  Avg.token   Fun Cnt  Warning cnt   Fun Rt   nloc Rt
------------------------------------------------------------------------------------------
        80       9.0     2.0       71.2        6            0      0.00    0.00
```

对文件中每个函数分别计算下面各项，location 反映每个方法的位置。

|         NLOC       | CCN    | token|  PARAM | length |
| :----------------: | :----: | :---: | :------: | :------: |
| 排除注释和空行的总行数 | 圈复杂度 | 符号数 | 参数数量 | 函数实际行数 |

### 1.4 降低复杂度方法

1. 模块化和函数分解：将复杂的函数或模块分解为更小的、功能更单一的子函数或模块。
2. 减少嵌套层级：避免过多的嵌套结构，比如嵌套的 if 语句、循环和 switch 语句。
3. 简化条件表达式：可以将复杂的条件表达式提取为单独的布尔变量或函数，使得条件逻辑更清晰。

## 2. 静态检查

代码静态检查是通过分析源代码本身，而不是通过执行代码来检测潜在的错误、bug 和不良风格。对于 C 项目，主要检查：

- 内存问题：内存泄漏、悬空指针、数组越界等
- 代码风格问题：缩进、括号、命名等
- 性能问题：不必要的循环等
- 代码逻辑问题：条件判断、循环控制等

C 语言的静态检查工具如下图：

![c check](/code_quality/cpp_sc.png)

以 cppcheck 为例，对于下面代码：

```c
// test.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
void foo() {
    int* ptr = malloc(10 * sizeof(int));  // 申请内存但未释放
    if (ptr == NULL) {
        return;
    }
    ptr[10] = 0;  // 数组越界
    free(ptr);
    ptr = NULL;
}
int main() {
    char* str = malloc(10);
    strcpy(str, "This is a long string");  // 缓冲区溢出
    printf("%s\n", str);
    foo();
    char* ptr = malloc(10 * sizeof(int));
    free(ptr);
    free(ptr);  // 重复释放内存
    return 0;
}
```

使用命令 `cppcheck test.c` 直接在终端输出结果，使用 `cppcheck test.c 2> output.txt` 输出到文本文件，结果如下：

```shell
test.c:9:8: error: Array 'ptr[10]' accessed at index 10, which is out of bounds. [arrayIndexOutOfBounds]
    ptr[10] = 0;  // 数组越界
       ^
test.c:15:12: error: Buffer is accessed out of bounds: str [bufferAccessOutOfBounds]
    strcpy(str, "This is a long string");  // 缓冲区溢出
           ^
test.c:14:15: note: Assign str, buffer with size 10
    char* str = malloc(10);
              ^
test.c:15:12: note: Buffer overrun
    strcpy(str, "This is a long string");  // 缓冲区溢出
           ^
test.c:20:5: error: Memory pointed to by 'ptr' is freed twice. [doubleFree]
    free(ptr);  // 重复释放内存
    ^
test.c:19:5: note: Memory pointed to by 'ptr' is freed twice.
    free(ptr);
    ^
test.c:20:5: note: Memory pointed to by 'ptr' is freed twice.
    free(ptr);  // 重复释放内存
    ^
test.c:21:5: error: Memory leak: str [memleak]
    return 0;
    ^
```

## 3. 编程规范

对于 C/C++ 项目，影响力较大的是 Google C++ Style Guide。

下面是一些较典型的规则：

1. 只有当函数只有 10 行甚至更少时才将其定义为内联函数。
2. 使用标准的头文件包含顺序可增强可读性，避免隐藏依赖。避免使用 UNIX 特殊的快捷目录（比如 `./xxx.h` 和 `../xxx.h`）。
    
    - 本文件相关头文件
    - C 库
    - C++ 库
    - 其他库 .h
    - 本项目所用 .h

3. 不允许使用变长数组和 `alloca()`
4. 尽可能用 `sizeof(varname)` 代替 `sizeof(type)`

MISRA C/C++，是由英国汽车产业软件可靠性协会（Motor Industry Software Reliability Association）提出的 C/C++ 语言开发标准，在**嵌入式**开发领域有较高认可度。

{{< admonition type=tip title="tip" open=true >}}
cppcheck + python 可以实现对于 misra 规则的检查，并集成到 VScode 中。
{{< /admonition >}}

## 参考

1. [详解圈复杂度](https://cloud.tencent.com/developer/article/1900402)
2. [OpenHarmony C 语言编程规范](https://gitee.com/openharmony/docs/blob/master/zh-cn/contribute/OpenHarmony-c-coding-style-guide.md)
3. [Google C++ 风格指南](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/)
4. [各大厂 C/C++ 编程规范](https://www.cnblogs.com/safe-rules/p/16037810.html)
5. [C++ 代码审查工具 Cppcheck 和 TscanCode](https://cloud.tencent.com/developer/article/1999973)
