# CMake 构建项目


## 1. CMake 基本构建

![img](/CMake/cmakeflow.png)

以下面结构的简单项目为例，子项目 test 中需要依赖一个测试框架 Unity, 一个只有头文件和源码的 log 库和一个由多个源文件和头文件组成的哈希表库 uthash。测试框架 Unity 提供了 CMakelists.txt。

```console
 .
├──  inc  
│  ├──  module1
│  └──  module2   
├──  src                  
│  ├──  module1
│  ├──  module2             
│  └──  main.c
└──  test             
   ├──  module1_test           
   ├──  module2_test         
   └──  third_party          
        ├──  unity
        ├──  uthash  
        └──  log.c          
```

顶层 cmakelists.txt 的基本元素如下：

```c
cmake_minimum_required(VERSION 3.20)
project(hello C)

# C 语言标准
set(CMAKE_C_STANDARD 99)
set(CMAKE_BUILD_TYPE Debug)

# 禁止在源目录编译以及修改
set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)
set(CMAKE_DISABLE_SOURCE_CHANGES ON)

# 不同模式下全局的编译选项
SET(CMAKE_CXX_FLAGS_DEBUG "-Wall -g")
SET(CMAKE_CXX_FLAGS_RELEASE "-O3")

# clangd 支持，查看编译命令，生成一个 compile_commands.json
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

add_executable(${PROJECT_NAME} src/main.c)
target_link_libraries(${PROJECT_NAME} PUBLIC m)  # 链接数学库

# 开启 ctest 功能，此选项需要在顶层 CMakeLists.txt 定义
include(CTest)

# 添加子项目
add_subdirectory(test)
```

配置 `test` 文件夹的子项目，需要新建一个 `test/CMakeLists.txt`，可以使用下面的示例：

```c
project(test_runner)

# Unity 
add_subdirectory(third_party/Unity)

# log.c .h 文件目录
include_directories(third_party/log.c/src/)

# uthash 源码目录
file(GLOB UTHASH "third_party/uthash/src/*.c")

add_executable(${PROJECT_NAME} test_runner.c ${UTHASH})

# 直接添加第三方项目的源文件
target_sources(${PROJECT_NAME} PRIVATE third_party/log.c/src/log.c third_party/log.c/src/log.h)

# 设置第三方项目的条件编译选项
target_compile_definitions(${PROJECT_NAME} PRIVATE LOG_USE_COLOR)

# 添加 ctest 测试
add_test(${PROJECT_NAME} ${PROJECT_NAME})
target_link_libraries(${PROJECT_NAME} unity)
```

- 对于像 Unity 一样使用 CMake 构建，且顶层有 CMakeList.txt 的库，可以直接使用 `add_subdirectory` 以子项目形式导入。
- 哈希库 uthash 只提供了一个文件夹，可以使用 file 函数使用通配符添加指定的文件保存到变量，建议使用 `CONFIGURE_DEPENDS`, 添加新文件时可以自动更新变量（CMake 3.12 起）。
    
    ```c
    file(GLOB UTHASH CONFIGURE_DEPENDS "third_party/uthash/src/*.c")
    ```
- 如果想直接添加额外某几个源文件参与构建，使用 `target_sources` 添加文件路径（CMake 3.13 起）。 

## 2. 相对路径定义

一般 CMake 工程将可执行文件产生在 build 路径，当 C/C++ 工程需要读写工程目录下的文件或者产生数据时，如果通过绝对路径寻找文件则丧失了可移植性。

这时可以在 cmake 中定义工作路径宏或者使用 configure_file 功能解决。

### 2.1 add_definitions

```c
add_definitions(-DPROJECT_ROOT="${CMAKE_CURRENT_SOURCE_DIR}")
```

在 C++ 程序中，执行下面的语句就能获取这个路径。

```cpp
string file_path = string(PROJECT_ROOT);
```

在 C 程序中，需要定义辅助宏解决。

```c
#include <stdio.h>

#define STR_HELPER(x) #x
#define STR(x) STR_HELPER(x)

int main() {
    printf("Project root: %s\n", STR(PROJECT_ROOT));
    return 0;
}
```

在 `STR_HELPER(x)` 中，# 将宏参数 x 转换为一个字符串字面量。

### 2.2 configure_file

此命令的功能是在构建时，将文件（一般是 .in 结尾）复制到另一个位置并修改 CMake 变量，一般用于在构建过程中动态生成头文件、配置文件、脚本文件。例如现在需要把 log 文件产生在工程目录的 test/log 文件夹。

```c
set(LOG_DIR "${CMAKE_SOURCE_DIR}/test/log")
configure_file(${CMAKE_SOURCE_DIR}/test/config.h.in ${CMAKE_SOURCE_DIR}/include/config.h @ONLY)
```

`config.h.in` 的内容

```cpp
#pragma once
#define LOG_DIR "@LOG_DIR@"
```

构建项目时，将在 include 目录中生成文件 config.h 并带有正确路径。

> 当打开 CMAKE_DISABLE_SOURCE_CHANGES 时，将无法使用 configure_file 功能。

## 3. 代码覆盖率

C/C++ 的代码覆盖率主要是利用 gcc 套件中的 gcov 组件产生，加入特殊选项编译后会产生 `*.gcno` 文件，运行测试后进一步产生 `*.gcda` 文件，二者结合源码，使用 `gcov ${program.gcno}` 命令生成覆盖率信息。一般需要进一步使用 lcov 或者 gcovr 产生可读性更强的报告。

> - lcov 是一个 perl 项目，Linux 平台开发使用较多，跨平台能力差，在 windows 上使用 gcc 编译的项目无法使用。
> - gcovr 是一个 Python 项目，资料较少，跨平台能力较好，能集成到 jenkins 产生代码覆盖率趋势。
> - OpenCppCoverage 是专门给 Windows 平台的代码覆盖率工具，运行依赖与 MSVC 编译过程中产生的 pdb 文件。

![gcov](/CMake/gcov.jpg)

### 3.1 Gcov 编译选项

这里参考 gcovr 的[官方文档](https://gcovr.com/en/stable/cookbook.html?highlight=cmake#out-of-source-builds-with-cmake)。编译时需要打开 gcc 选项 `--coverage` （此项等同于 `-fprofile-arcs -ftest-coverage`），同时要禁用优化选项。

```c
set(CMAKE_C_FLAGS "-g -O0 -Wall --coverage")
set(CMAKE_CXX_FLAGS "-g -O0 -Wall --coverage")
...
add_compile_options(--coverage)
add_link_options(--coverage)
```

由于代码覆盖率不仅要看测试代码，还要看被测代码，因此编译选项需要全局打开。

### 3.2 VSCode Gcov View

在 VSCode 中可以借助插件 Gcov View 显示代码覆盖情况。
生成 .gcda 文件后，Ctrl + shift + p 输入 "Gcov Viewer"，选择 show 即可看到哪些行被覆盖。

![vscode coverage](/CMake/cmake_coverage.png)

使用过程中可能会发现如下执行错误：

![Gcov show error](/CMake/Gcov_show_error.png)

这种情况下使用 `gcov xxx.gcno` 也会出现错误：

![gcno error](/CMake/gcno_error.png)

这是由于命令行调用的 gcov 版本和编译使用的 gcc 版本不一致导致的。尤其是在使用 cmake 插件没有规定编译工具链版本时容易出现此类问题。

### 3.3 Gcovr

可以使用 pip 安装，要确保 python 安装的可执行文件添加到了环境变量中。

```shell
pip install gcovr
```

由于 CMake 源码和编译文件是分离的，产生报告的命令为：

```shell
> gcovr -r $SRC_Path $BUILD_Path
```

例如 CMake 编译产生的文件夹是 build，build 目录和 src 目录在同一级，执行：

```shell
cd build
gcovr -r ../ .
```

程序会递归扫描文件夹下所有 `.gcno` 文件。输出结果类似于：

```shell
------------------------------------------------------------------------------
                           GCC Code Coverage Report
Directory: ../
------------------------------------------------------------------------------
File                                       Lines    Exec  Cover   Missing
------------------------------------------------------------------------------
src/example/crc.c                              6       6   100%   
src/example/example.c                          2       0     0%   3,5
src/main.c                                     6       6   100%   
test/example/crc_test.c                       27      27   100%   
test/example/main_test.c                       5       5   100%   
```

如果要产生 html 报告，并设置标题，排除一些文件夹，需要添加参数：

```shell
gcovr -r ../ . --html --html-details --html-title "report name" -e ../test/ -o report.html
```

总体：

![gcovr1](/CMake/gcovr1.png)

细分：

![gcovr2](/CMake/gcovr2.png)

进一步可以产生 xml 格式报告[集成到 Jenkins](https://blog.csdn.net/sinat_39037205/article/details/122930653) 中。

## 4. 常用配置项

### 4.1 项目问题检查 Sanitizer

gcc/clang 集成了 Sanitizer 内存检查工具。包括：

- AddressSanitizer: a memory error detector
- LeakSanitizer: a memory leak detector
- MemeorySanitizer: a detector of uninitialized reads
- ThreadSanitizer: a data race and deadlock detector
- UndefinedBehaviorSanitizer: a undefined behavior detector

开启需要在 CMakeLists.txt 中定义：

```c
set(CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -fsanitize=address -fno-omit-frame-pointer 
                                    -fsanitize=undefined -fsanitize=leak 
                                    -fsanitize=signed-integer-overflow")

set(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -fsanitize=address -fno-omit-frame-pointer 
                                    -fsanitize=undefined -fsanitize=leak 
                                    -fsanitize=signed-integer-overflow")


set(CMAKE_EXE_LINKER_FLAGS  "${CMAKE_EXE_LINKER_FLAGS} -fsanitize=address -fno-omit-frame-pointer 
                                    -fsanitize=undefined -fsanitize=leak 
                                    -fsanitize=signed-integer-overflow")

...

# 下面一行实际环境中不一定需要开启，一般 gcc 自动支持
target_link_libraries(${PROJECT_NAME} asan)
```

运行后，输出结果额外带有内存泄漏位置的调用栈，顺着查找即可。以下是一些常用的参数：

1. `-fsanitize=address`：启用 AddressSanitizer（ASan）
2. `-fsanitize=leak`：启用 LeakSanitizer，用于检测内存泄漏。
3. `-fsanitize=thread`：启用 ThreadSanitizer，用于检测并发程序中的数据竞争和死锁。
4. `-fsanitize=memory`：启用 MemorySanitizer，用于检测对未初始化内存的访问，有助于提高内存安全性。
5. `-fsanitize=signed-integer-overflow`：用于检测整数溢出。
6. `-fno-omit-frame-pointer`：告诉编译器不要优化掉函数调用的框架指针。在进行调试或性能分析时，框架指针对于跟踪函数调用关系非常重要。
7. `-fsanitize=undefined`：启用 UndefinedBehaviorSanitizer（UBSan），它用于检测一些未定义行为。比如整数溢出、除以零等。

可以将以上选项提取到项目的 `cmake/SanitizerOptions.cmake` 文件中。

```c
macro(set_sanitizer_options)
    set(CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -fsanitize=address -fno-omit-frame-pointer 
                                        -fsanitize=undefined -fsanitize=leak 
                                        -fsanitize=signed-integer-overflow")
    set(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -fsanitize=address -fno-omit-frame-pointer 
                                        -fsanitize=undefined -fsanitize=leak 
                                        -fsanitize=signed-integer-overflow")
    set(CMAKE_EXE_LINKER_FLAGS  "${CMAKE_EXE_LINKER_FLAGS} -fsanitize=address -fno-omit-frame-pointer 
                                        -fsanitize=undefined -fsanitize=leak 
                                        -fsanitize=signed-integer-overflow")
endmacro()
```

在 CMakeLists.txt 中 include。

```c
# 选项开关
option(USE_SANITIZER "Use Sanitizer" OFF)
...
# 导入额外编译选项
if(USE_SANITIZER)
    include(cmake/SanitizerOptions.cmake)
    set_sanitizer_options()
endif()
```

### 4.2 保留预处理后文件

添加编译选项：

```c
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -save-temps=obj")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -save-temps=obj")
```

### 4.3 输出 map 文件

输出 .map 文件到 build 目录，添加链接选项：

```c
set(MAP_FILE ${CMAKE_BINARY_DIR}/${PROJECT_NAME}.map)
set(CMAKE_EXE_LINKER_FLAGS "-Wl,-Map=${MAP_FILE}")
```

### 4.4 设置输出目录

`CMAKE_ARCHIVE_OUTPUT_DIRECTORY`、`CMAKE_LIBRARY_OUTPUT_DIRECTORY` 和 `CMAKE_RUNTIME_OUTPUT_DIRECTORY` 变量用于指定构建过程中生成的静态库、动态库和可执行文件的输出路径。`CMAKE_INSTALL_LIBDIR` 和 `CMAKE_INSTALL_BINDIR` 是预定义的变量。

```c
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_BINDIR})
```

### 4.5 启用调试中宏打印

```c
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g3 -O0 -Wall")
# 也有写成
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -gdwarf-2 -g3 -O0 -Wall")
```

### 4.6 开启 LTO

LTO（Link Time Optimization）是一种编译优化技术，它在链接阶段对整个程序进行优化，而不仅仅是在单独编译单元的优化。可能会增加编译时间和内存占用。可以通过变量 `CMAKE_INTERPROCEDURAL_OPTIMIZATION`（CMake 3.9+ 可用）或对目标指定 `INTERPROCEDURAL_OPTIMIZATION` 属性来打开它。

```c
# 设置 Release 模式下启用 LTO 编译选项
if(CMAKE_BUILD_TYPE STREQUAL "Release")
    include(CheckIPOSupported)
    check_ipo_supported(RESULT result)
    if(result)
        message(STATUS "LTO is supported")
        set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
    else()
        message(WARNING "LTO is not supported")
    endif()
endif()

# 为特定目标设置 LTO
include(CheckIPOSupported)
check_ipo_supported(RESULT result)
if(result)
  set_target_properties(foo PROPERTIES INTERPROCEDURAL_OPTIMIZATION TRUE)
endif()
```

### 4.7 地址无关代码 (Position independent code)

动态链接库必须。一般用标志 -fPIC 来设置。大部分情况下，不需要去显式声明，CMake 将会在 SHARED 以及 MODULE 类型的库中自动的包含此标志。如果需要显式的声明：

```c
# 对某个目标设置 fPIC 标志
set_target_properties(lib1 PROPERTIES POSITION_INDEPENDENT_CODE ON)
```

## 5. Arm 项目和 x86 项目构建上的区别

最典型的例子，char 的符号性并没有明确定义，取决于编译器实现。ARM 架构的 LDRSB 指令，可以将一个有符号字节加载到 32 位寄存器并进行符号扩展。但是，这个指令不是 ARM 架构最早版本的一部分。当时为了避免符号扩展的开销，arm 平台的 char 默认是无符号类型。

在 CMake 中通过设置 gcc 的编译选项来调整编译时的符号性。

```c
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fsigned-char")
```

除此之外构建跨平台项目需要注意：

- 浮点数处理：ARM 有硬浮点和软浮点两种模式，可以通过 -mfloat-abi 选项来选择。
- 指令集版本：ARM 有多个版本的指令集，例如 ARMv7、ARMv8 等，通过 -march 选项来指定目标架构。
- 内存模型：ARM 支持多种内存模型，包括小端（little-endian）和大端（big-endian）。通过 -mlittle-endian 或 -mbig-endian 选项来选择。x86 平台默认使用小端模型。
- ABI（Application Binary Interface）：不同的 ARM ABI（如 AAPCS、AAPCS-VFP 等）可能影响函数调用约定、数据对齐等。通过 -mabi 选项来选择。

## 6. VSCode 集成 Vcpkg 包管理

安装 vcpkg 并进行如下必要的设置：

```bash
git clone https://github.com/microsoft/vcpkg.git
./bootstrap-vcpkg.sh
cd vcpkg
./vcpkg integrate bash # or zsh
./vcpkg install fmt
```

在工作区的 settings.json 中添加 CMAKE_TOOLCHAIN_FILE：

```json
{
    "cmake.configureSettings": {
        "CMAKE_TOOLCHAIN_FILE": "[vcpkg root]/scripts/buildsystems/vcpkg.cmake"
    }
}
```

也可以直接在 CMakeLists.txt 中添加：

```c
set(CMAKE_TOOLCHAIN_FILE "[vcpkg root]/scripts/buildsystems/vcpkg.cmake")
```

重新加载工作区后就能使用 vcpkg 安装的包：

```c
find_package(fmt CONFIG REQUIRED)
...
target_link_libraries(${PROJECT_NAME} PRIVATE fmt::fmt-header-only)
```

## 7. CMake Presets

当项目越来越复杂，需要考虑跨系统、架构、编译器以及测试各种编译选项时，cmake 就会存在许多需要设置的选项，具体表现为每构建一次，就需要输入一堆 `-D` 选项，同时 CMakeLists.txt 中的内容会变冗余繁杂。

CMake 3.19 后更新的预设功能，可以通过**项目顶层**的 CMakePresets.json 和 CMakeUserPresets.json 文件，构成配置矩阵，简化 CMakeLists.txt 和参数输入。

以一个 hello world 项目举例，通过 option `USE_SDL` 控制是否生成图形界面版本，否则生成命令行版本。编译时使用包管理器安装的 gcc-11 和**不在环境变量中的 gcc-13**。下面是 CMakePresets.json 的内容：

```json
{
    "version": 3,
    "configurePresets": [
    {
        "name": "ci-base",
        "displayName": "base config",
        "generator": "Unix Makefiles",
        "binaryDir": "build",
        "cacheVariables": {
            "CMAKE_C_STANDARD": "99",
            "CMAKE_CXX_STANDARD": "20"
        },
        "hidden": true
    },
    {
        "name": "ci-debug",
        "inherits": "ci-base",
        "cacheVariables": {
            "CMAKE_BUILD_TYPE": "Debug",
            "CMAKE_CXX_FLAGS": "-g"
        }
    },
    {
        "name": "ci-release",
        "inherits": "ci-base",
        "cacheVariables": {
            "CMAKE_BUILD_TYPE": "Release",
            "CMAKE_CXX_FLAGS": "-O3"
        }
    }
    ]
}
```

CMakePresets.json 中编写通用配置，也可以用于 CI 构建。上面的内容定义了项目的通用配置，"inherits" 用于继承其他字段的定义，**可以继承多个字段**。通过 "hidden" 隐藏不必要的组件字段。

CMakeUserPresets.json 通常由开发人员定义特殊配置，在自己的构建环境中使用，因此不应将其置于版本控制中。

> 应该从名称上区分项目预设还是自定义预设。可以在项目定义预设加上 "ci-" 前缀。

CMakeUserPresets.json 的内容如下：

```json
{
    "version": 3,
    "configurePresets": [
    {
        "name": "gcc-12",
        "hidden": true,
        "cacheVariables": {
            "CMAKE_C_COMPILER": "gcc-12",
            "CMAKE_CXX_COMPILER": "g++-12"
        }
    },
    {
        "name": "gcc-13",
        "hidden": true,
        "cacheVariables": {
            "CMAKE_C_COMPILER": "/opt/gcc-13.2.0/bin/gcc",
            "CMAKE_CXX_COMPILER": "/opt/gcc-13.2.0/bin/g++",
            "CMAKE_INSTALL_RPATH": "/opt/gcc-13.2.0/lib64",
            "CMAKE_BUILD_WITH_INSTALL_RPATH" : true
        }
    },
    {
        "name": "gui app debug",
        "inherits": ["ci-debug", "gcc-13"],
        "binaryDir": "${sourceDir}/build/gui",
        "cacheVariables": {
            "CMAKE_CXX_FLAGS": "-Wall",
            "USE_SDL":true
        }
    },
    {
        "name": "console app debug",
        "inherits": ["ci-debug", "gcc-12"],
        "binaryDir": "${sourceDir}/build/console",
        "cacheVariables": {
            "CMAKE_CXX_FLAGS": "-Wall",
            "USE_SDL":false
        }
    }
    ],
    "buildPresets" :[
    {
        "name": "gui app debug",
        "configurePreset": "gui app debug"
    },
    {
        "name": "console app debug",
        "configurePreset": "console app debug"
    }
    ]
}
```

CMakeUserPresets.json 可以直接继承 CMakePresets.json 中的字段。注意此处的 "CMAKE_CXX_FLAGS" 字段并不会覆盖 CMakePresets.json 中的 "CMAKE_CXX_FLAGS" 设置，而是在原来的基础上添加。

每个 "buildPresets" 需要配合一个同名 "configurePreset" 完成构建，此时可以查看 cmake 预设：

```bash
➜  cmake --list-presets
Available configure presets:

  "gui app debug"
  "console app debug"
  "ci-debug"
  "ci-release"
```

如果此时要构建 gui 版本，则输入：

```bash
cmake --preset "gui app debug"
cmake --build --preset "gui app debug"
```

如果使用 VSCode 插件则无需输入命令行，直接鼠标点选即可构建。

## 8. rpath

全名 Run-time Search Path，运行时链接器搜索路径。这个值硬编码在可执行文件或者动态库的 ELF 结构的 .dynamic 节中。动态库搜索路径的顺序：

- rpath
- LD_LIBRARY_PATH
- RUNPATH
- ldconfig
- 内置路径 /lib、/usr/lib

使用 gcc 编译程序时，`-L` 指定编译时搜索动态库的位置，选项 `-Wl,-rpath` 指定程序运行时搜索动态库的路径。

在 CMake 中主要由下面几个变量控制：

- CMAKE_INSTALL_RPATH 程序安装后，搜索 dynamic library 路径
- CMAKE_BUILD_WITH_INSTALL_RPATH 设为 True 时默认使用 CMAKE_INSTALL_RPATH
- CMAKE_BUILD_RPATH_USE_ORIGIN 使用 $origin 相对路径

在 CMake 构建完成程序会带有 rpath，但如果程序需要考虑安装问题时需要设置此项。

另一种使用场景是使用自己编译的工具链版本构建程序时，添加链接路径，假设 gcc-13 被安装在了路径 `/opt/gcc-13.2.0/`：

```c
cmake_minimum_required(VERSION 3.20)
project(MyProject)
# 检查编译器版本是否是 gcc-13
if (CMAKE_CXX_COMPILER_ID STREQUAL "" AND CMAKE_CXX_COMPILER_VERSION VERSION_EQUAL "13.2.0")
    message(STATUS "Detected GCC 13 compiler")
    set(CMAKE_INSTALL_RPATH "/opt/gcc-13.2.0/lib64")
    set(CMAKE_BUILD_WITH_INSTALL_RPATH true)
endif()

# 添加源文件和可执行文件
add_executable(MyExecutable main.cpp)

# 安装目标
install(TARGETS MyExecutable DESTINATION bin)
```

这样程序运行时就不会出现各种 `'GLIBC_2.xx' not found` 的问题。

## 9. 编译选项控制

target_compile_options 命令用于给特定构建目标添加编译选项。可以添加三个级别的可见性：INTERFACE、PUBLIC 和 PRIVATE。定义如下：

* PRIVATE，编译选项会应用于给定的目标，不会传递给与目标相关的目标。
* INTERFACE，给定的编译选项将只应用于指定目标，并传递给与目标相关的目标。
* PUBLIC，编译选项将应用于指定目标和使用它的目标。

例如：

```c
list(APPEND flags "-fno-exceptions" "-fno-rtti")
add_library(xlib STATIC xxx.cpp)
target_compile_options(xlib PRIVATE ${flags})
```

> 1. 检查对每个源文件的编译选项，除了设置 CMAKE_EXPORT_COMPILE_COMMANDS 外，还可以通过 `cmake --build . -- VERBOSE=1`
>
> 2. 选项 CMAKE_EXPORT_COMPILE_COMMANDS 仅当使用 Makefile 和 Ninja 生成器时有效。

## 10. Clang 静态分析器

安装 clang 和 llvm 套件，安装 clang static analyzer：

```bash
sudo apt install clang-tools
scan-build -h
```

在执行 cmake 和 make 命令前加上 `scan-build`，将会替换 CC 和 CXX 环境变量。

```bash
scan-build cmake ..
scan-build make
```

如果找到错误会有类似提示：

```
scan-build: Analysis run complete.
scan-build: 2 bugs found.
scan-build: Run 'scan-view /tmp/scan-build-2024-04-30-151415-5388-1' to examine bug reports.
```

执行提示的 `scan-view xxx` 可以在浏览器上查看错误详情。

## 11. CMake 行为准则

### 11.1 CMake 应避免的行为

- 不要使用具有全局作用域的函数：这包含 `link_directories`、 `include_libraries` 等相似的函数。
- 不要添加非必要的 PUBLIC 要求：你应该避免把一些不必要的东西强加给用户（-Wall）。相比于 PUBLIC，更应该把他们声明为 PRIVATE。
- 不要在 file 函数中添加 GLOB 文件：如果不重新运行 CMake，Make 或者其他的工具将不会知道你是否添加了某个文件。值得注意的是，CMake 3.12 添加了一个 CONFIGURE_DEPENDS 标志能够使你更好的完成这件事。
- 将库直接链接到需要构建的目标上：如果可以的话，总是显式的将库链接到目标上。
- 当链接库文件时，不要省略 PUBLIC 或 PRIVATE 关键字：这将会导致后续所有的链接都是默认的。

### 11.2 CMake 应遵守的规范

- 把 CMake 程序视作代码：它是代码。它应该和其他的代码一样，是整洁并且可读的。
- 建立目标的观念：你的目标应该代表一系列的概念。为任何需要保持一致的东西指定一个（导入型）INTERFACE 目标，然后每次都链接到该目标。
- 导出你的接口：你的 CMake 项目应该可以直接构建或者安装。
- 为库书写一个 Config.cmake 文件：这是库作者为支持客户的体验而应该做的。
- 声明一个 ALIAS 目标以保持使用的一致性：使用 `add_subdirectory` 和 `find_package` 应该提供相同的目标和命名空间。
- 将常见的功能合并到有详细文档的函数或宏中：函数往往是更好的选择。
- 使用小写的函数名：CMake 的函数和宏的名字可以定义为大写或小写，但是一般都使用小写，变量名用大写。
- 使用 cmake_policy 和/或 限定版本号范围：每次改变版本特性 (policy) 都要有据可依。应该只有不得不使用旧特性时才降低特性 (policy) 版本。

## 参考

1. [Communication with your code](https://cliutils.gitlab.io/modern-cmake/chapters/basics/comms.html)
2. [在 CMakeLists.txt 中添加源文件的几种方法](https://zhuanlan.zhihu.com/p/631113006)
3. [CMake - 使用 target_sources() 提高源文件处理能力](https://blog.csdn.net/guaaaaaaa/article/details/125601766)
4. [Using AddressSanitizer (ASan) in a CMake project](https://felsoci.sk/blog/using-address-sanitizer-asan-in-a-cmake-project.html)
5. [unsigned char and signed char](https://developer.arm.com/documentation/den0013/d/Porting/Miscellaneous-C-porting-issues/unsigned-char-and-signed-char)
6. [cmake：这样查看源码的预编译信息](https://zhuanlan.zhihu.com/p/596350807)
7. [The GDB developer's GNU Debugger tutorial, Part 1: Getting started with the debugger](https://developers.redhat.com/blog/2021/04/30/the-gdb-developers-gnu-debugger-tutorial-part-1-getting-started-with-the-debugger#compiler_options)
8. [使用 Gcov 和 LCOV 度量 C/C++ 项目的代码覆盖率](https://zhuanlan.zhihu.com/p/402463278)
9. [Linux 下 gcovr 的使用细节](https://blog.csdn.net/whahu1989/article/details/121711957)
10. [Building with CMake Presets](https://mcuoneclipse.com/2023/12/03/building-with-cmake-presets/)
11. [使用 CMake Presets](https://hedzr.com/c++/algorithm/using-cmake-presets/)
12. [CMake Presets](https://blog.feabhas.com/2023/08/cmake-presets/)
13. [Organizing CMake presets](https://dominikberner.ch/cmake-presets-best-practices/)
14. [Understanding RPATH (With CMake)](https://duerrenberger.dev/blog/2021/08/04/understanding-rpath-with-cmake/)
15. [链接选项 rpath 的应用和原理](https://bewaremypower.github.io/2020/07/14/%E9%93%BE%E6%8E%A5%E9%80%89%E9%A1%B9-rpath-%E7%9A%84%E5%BA%94%E7%94%A8%E5%92%8C%E5%8E%9F%E7%90%86/)
16. [RPATH 简介以及 CMake 中的处理](https://blog.xizhibei.me/2021/02/12/a-brief-intro-of-rpath/)
17. [rpath 使用说明](https://zhuanlan.zhihu.com/p/661388872)
18. [【ToolChains】| CMake 技巧](https://www.cnblogs.com/RioTian/p/17869507.html)
19. [LLVM 之 Clang 静态分析器篇（1）：如何使用 Clang 静态分析器](https://csstormq.github.io/blog/LLVM%20%E4%B9%8B%20Clang%20%E9%9D%99%E6%80%81%E5%88%86%E6%9E%90%E5%99%A8%E7%AF%87%EF%BC%881%EF%BC%89%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%20Clang%20%E9%9D%99%E6%80%81%E5%88%86%E6%9E%90%E5%99%A8)
20. [CMake 行为准则 (Do's and Don'ts)](https://modern-cmake-cn.github.io/Modern-CMake-zh_CN/chapters/intro/dodonot.html)

