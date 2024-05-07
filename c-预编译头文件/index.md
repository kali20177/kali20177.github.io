# C++ 预编译头文件


## 1. 预编译头文件

将头文件编译成中间格式，从而节省在开发过程中编译器从头编译造成的开销。

![compile_time_joke](/Cpp_precom/compile_time_joke.png)

一般预编译头文件中的头文件是**标准库，操作系统 API 或者无需修改的开发框架**，通过预编译减少编译时间。

在 Vistual Studio 中，项目模板的预编译头文件是 `pch.h`或`stdafx.h`。项目文件夹下生成的`ipch`文件夹是存放预编译头结果的地方，故上传 github 时可直接忽略。

## 2. g++ 中使用预编译头

以一个简单项目为例：

```text
src
 | -- main.cpp
 | -- pch,h
 | -- pch.cpp
```

```cpp
// pch.h 文件内容
#pragma once

#include <iostream>
#include <functional>
#include <algorithm>
#include <utility>

#include <stack>
#include <deque>
#include <string>
#include <vector>
#include <unordered_map>
```

在不使用预编译头的情况下测试：

```bash
time g++ -std=c++11 main.cpp
```

![bmg](/Cpp_precom/no_pre_compiler_header.png)

使用预编译头，需要先生成中间文件

```bash
g++ -std=c++11 pch.h
```

![bmg](/Cpp_precom/create_pch.png)

之后会生成`pch.h.gch`文件

再执行编译，得到：

![bmg](/Cpp_precom/g++_pre_compiler_header.png.png)

## 3. 在 Vistual Studio 中使用预编译头

还是针对以上项目。选中想要创建预编译头的 `pch.cpp` 文件，右键属性 --> C/C++ --> 预编译头 --> 创建。

![bmg](/Cpp_precom/vs_precom1.png)

然后配置整个项目的属性 --> C/C++ --> 预编译头 --> 使用。

![bmg](/Cpp_precom/vs_precom2.png)

这样，经过第一次编译后，在之后的开发过程中能极大减少编译时间。

## 4. CMake 中使用预编译头

此功能在 CMake 3.16 版本之后被支持。在 CMake 中使用预编译头需要使用 `target_precompile_headers` 命令。假设有一个 main.cpp 和一个 mypch.h 的预编译头文件。

```cpp
// mypch.h
#include <iostream>
#include <string>
#include <vector>
```

那么 CMake 文件：

```c
cmake_minimum_required(VERSION 3.16)
project(PrecompiledHeadersExample)
add_executable(MyApp main.cpp)

# 指定预编译头文件
target_precompile_headers(MyApp PRIVATE mypch.h)
```

> 对于 C 和 C++ 混合项目，最好单独编译成不同的库或者可执行文件，不要出现 add_executable 阶段 .cpp 和 .c 一块儿被编译的情况。这会导致 CMake 无法正确处理预编译头。

