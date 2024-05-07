# 单元测试框架


![unit integration testing](/UnitTesting/unit-integration-testing.png)

## 1. 基本概念

单元测试是针对软件设计的最小单位进行测试，这里的最小单位可以是函数、模块或者类。目的是检查每个程序单元能否正确实现详细设计说明中的模块功能、性能、接口和设计约束等要求，发现各模块内部可能存在的各种错误。

主要采用**白盒测试**保证单元的最大覆盖率，发现编码和设计中的错误。单元测试一般在**宿主机**中运行[4]。

![img](/UnitTesting/whitebox.png)

### 1.0 单元

什么是单元，要不要从最小单元开测，不同人的观点都不相同。

笼统的说法：单元测试是针对软件设计的最小单位进行测试，这里的最小单位可以是函数、模块或者类。

《单元测试的艺术》

![SUT](/UnitTesting/SUT.png)

《[Google 软件工程](https://qiangmzsx.github.io/Software-Engineering-at-Google/#/zh-cn/Chapter-12_Unit_Testing/Chapter-12_Unit_Testing)》

![Google test](/UnitTesting/Google_test.png)

> 什么是 "公共 API "并不总是很清楚，这个问题实际上涉及到单元测试中的 "单元"的核心。单元可以小到一个单独的函数，也可以大到由几个相关的包/模块组成的集合。当我们在这里说 "公共 API"时，我们实际上是在谈论该单元暴露给拥有该代码的团队之外的第三方的 API。
>
> 对代码实现细节进行调用的测试是脆弱的，几乎任何对被测系统的重构（例如重命名其方法、将其分解为辅助类或更改序列化格式）都会导致测试中断，即使此类更改对类的实际用户是不可见的。

### 1.1 单元测试名词

- **断言（Assertions）** 是检查条件是否为真的语句。断言的结果可能是*成功*或者*失败*，而失败又分为*非致命失败*或*致命失败*。如果发生*致命失败*，测试进程将中止当前运行，否则它将继续运行。类似函数调用的**宏**。
  
  一般包含：

  - bool 判断
  - 逻辑运算判断
  - 字符串判断
  - 浮点数判断
  - 异常判断

- **测试（Test）** 使用断言来验证被测试代码的行为。如果测试崩溃或断言失败，则测试失败；否则测试成功。

- **测试套件（Test Suite）** 包含一个或多个**测试（Test）**。当测试套件中的多个测试需要共享通用对象和子例程时，可以将它们放入**测试夹具（Test Fixture）**。

- **测试夹具** 多个共有的数据配置的测试群 (包括初始化 Setup、运行 Run、验证结果 validate、清理 TearDown 四个过程)

  ![img](/UnitTesting/Test_fixture.png)

- TAP 测试任意协议（Test Anything Protocol，TAP）是一种用于记录和报告测试结果的文本协议。它是一种轻量级的标准，旨在为不同编程语言的测试框架提供一种通用的格式，使测试结果能够被解析和分析，从而方便自动化测试的执行和报告。
  
    > 1. 通用性：TAP 不限于特定的编程语言或测试框架，因此可以用于多种语言的测试。因此，可以使用 TAP 来记录 Perl、Python、Ruby、C、Java 等各种语言的测试结果。
    > 2. 简洁的格式：TAP 使用简单的文本格式，易于阅读和理解。每个测试用例都表示为一行文本，其中包含测试结果、测试说明和其他信息。
    > 3. 易于解析：TAP 的格式易于解析，可以使用脚本或工具来分析测试输出并生成有关测试结果的报告。这种特性使 TAP 在自动化测试流程中特别有用。
    > 4. 丰富的测试结果表示：TAP 可以表示测试成功、失败、跳过、计划的测试数量等。

- 参数化框架 参数化测试是软件测试中的一种方法，它允许你在单个测试用例中运行多组测试数据，以便更全面地验证代码的不同输入情况。参数化测试框架是为了简化参数化测试的实现而设计的工具或库，它可以自动化地运行相同测试逻辑的不同参数组合，从而减少重复代码的编写，提高测试效率，并且更易于维护。
    > 一般情况下，参数化测试框架主要用于在相同测试逻辑下，对不同的输入数据进行测试。如果不同的测试数据需要调用不同的函数，并且这些函数的参数类型和数量都不同，那么通常参数化测试框架可能无法直接满足这种需求。
    > 参数化测试框架在运行时会根据预定义的参数组合，调用同一个测试函数，并传递不同的参数值。这个测试函数通常是事先定义好的，而且参数类型和数量是固定的。如果你需要在不同的测试数据情况下调用不同的函数，并且这些函数的参数类型和数量都不同，可能需要考虑其他的测试组织方式。

### 1.2 双目标开发

- 目标设备
- 宿主机（首要）

目标机上的测试会花费更长的时间和成本，基于宿主机的测试代价较小，先在软件代码层面排除所有问题。避免同时修改硬件和软件 bug，防止硬件部和软件部相互扯皮。并且能更容易移植到其他平台[1]。

定时问题有关的白盒测试、中断测试、硬件接口测试只能在目标机运行，在开发周期中处于靠后位置[4]。

![img](/UnitTesting/最终测试.jpg)

### 1.3 Mock

在进行单元测试时，我们想要测试模块 A，但是模块 A 却依赖于模块 B，当模块 B 无法满足预期时就无法对模块 A 进行测试，主要由于下面几个原因：

> - **模块 B 依赖于硬件设备**
> - 真实的模块 B 的返回值无法满足我们的预期
> - **团队开发中模块 B 尚未实现**
> - 如何比较方便的控制模块 B 的行为，返回特定数据

这时就需要对函数 B 进行打桩，使其达到我们预期的效果。

  ![img](/UnitTesting/MO.png)

生成测试桩方式：

- 返回固定值
- 模拟真实的下层模块（虚拟机之类）
- 仿制对象（mock）

> **开发一个测试桩的工作量和开发一个实际的模块的工作量是相当的[1]。一般开发中选择第三种**

Mock 是为了解决不同的单元之间由于耦合而难于开发、测试的问题，Mock 既能出现在单元测试中，也会出现在集成测试、系统测试过程中。Mock 最大的功能是帮你把单元测试的耦合分解开，如果你的代码对另一个类或者接口有依赖，它能够帮你模拟这些依赖，并帮你验证所调用的依赖的行为。

**Fake、Stub 和 Mock** 的区别：

[伪对象、桩对象、模拟对象 | 单元测试](https://blog.csdn.net/yufaxingxing/article/details/113713433)

- Fake 译作**伪对象**，生产环境下具体实现的简化版本的对象，可以指 Stub 也可以指 Mock。
- Stub 译作**测试桩**或**存根**，在当前测试用例的指示下返回某些值。
- Mock 译作**仿制对象**或模拟对象，校验函数调用、调用顺序，以及从被测试代码传向依赖组建的参数，还可以设定向被测代码返回指定的值，常用于处理有多个复杂交互的场景。

  ![img](/UnitTesting/mock.png)

  **三者概念不必做严格区分**。

对 C 语言进行 mock 的三种方法：函数指针、预处理时期条件编译替换、编译后 Link 替换[5]。

三种测试方法分别对应的场景[3]：

![img](/UnitTesting/mock选择.png)

所以最好在一开始选择支持 Mock 对象的测试框架。

## 2. 需求预估

- Unit Test
- 硬件对象模拟（mock），并且由于我们有 HAL 层，更容易做到隔离

## 3. 通用工具

- 代码覆盖率工具
  - gcovr / lcov —— linux [github CMake codecov](https://github.com/RWTH-HPC/CMake-codecov)
  - [OpenCppCoverage](https://github.com/OpenCppCoverage/OpenCppCoverage) —— windows

  ![lcov](/UnitTesting/lcov.png)

- 代码复杂度分析 (圈复杂度)

  - [lizard](https://github.com/terryyin/lizard)

- 静态检查类工具
  - cppcheck  
  - splint
  - clang-tidy
  - [flawfinder](https://github.com/david-a-wheeler/flawfinder)

- 代码格式化
  - clang-format

- ctest

  > `CTest` 是 CMake 集成的一个测试工具，在使用 CMakeLists.txt 文件编译工程的时候，CTest 会自动 configure、build、test 和展现测试结果。能执行一些**简单**测试，并集成 python 和 shell 脚本。
  >
  > - `enable_testing()`，测试这个目录和所有子文件夹。
  >
  > - `add_test()`，定义了一个新的测试，并设置测试名称和运行命令。
  >
  > **在使用测试框架的情况下，这个选项不是必须开启的，但是当测试框架产生的可执行文件较多时，`CTest` 可以执行批量测试并展示结果。**

- git hooks 服务器端自动执行测试，持续集成工具

- 脚本生成测试项目，使用 TUI 程序选择模块测试，类似编译 linux kernal 时的 menuconfig

  ![img](/UnitTesting/menuconfig.png)

  [脚本生成 CMake + Unity 项目](https://github.com/bredlej/init-cmake-project/blob/main/init-cmake-project)
  
## 4. 框架对比

|        名称        | on mcu | 参考 & 使用 | 维护 |   行业    | mock | 易用性 |  结果 |
|:------------------:|:------:|:--------:|:----:|:---------:|:----:|:----:|:----:|
|      CppUTest      |   ☑️    |  🔥🔥   |      | 博世、AWS |  ✅   |  ✅  |    |
|       gtest        |   ☑️    |  🔥🔥🔥  |      |   Google  |  ✅   |  ✅ |    |
|       Unity        |   ✅   |   🔥🔥   |      |           |  ✅   |  ✅ |    |
|       cmocka       |   ☑️    |    🔥     |      |   小米    |  ✅   |  ✅  |   |
|       CUnit        |   ✅   |    🔥    |       |           |       |      |   |
|     criterion      |        |           |      |           |       |  ✅  |   |
|     libcester      |   ☑️    |    ❌     |       |          |  ✅   |      |   |
|       cutest       |   ✅   |    ❌     |  ❌   |          |       |      |   |
|       cgreen       |        |           |       |          |  ✅    |     |    |
| PARASOFT C/C++TEST |        |  商业软件   |        |          |       |      |   |
|    GNU Autounit    |        |    ❌     |        |          |       |  ❌ |   |
|    EmbeddedUnit    |   ✅   |    ❌     |  ❌   |          |       |      |   |
|      Cmockery      |        |    ❌     |  ❌   |          |       |  ❌  |   |
|      AceUnit       |   ✅   |    ❌     |        |          |       |     |   |
|       Check        |        |           |   ❌   |          |       |      |   |

> ☑️ 表示能运行在性能较强的嵌入式平台 (包含操作系统)； ✅ 表示能运行在性能较差的嵌入式平台

[完整列表在此处](https://en.wikipedia.org/wiki/List_of_unit_testing_frameworks#C) 已经删除一些过于小众的测试框架。

## 5. 框架模板

使用 cmake 方式，给表上靠前的框架构建单元测试模板，统一做 CRC 校验测试。

### 5.1 CppUTest

[CppUTest 官方](https://cpputest.github.io/)

CppUTest 是一个基于 C/C++ 的单元 xUnit 测试框架，用于单元测试和代码测试驱动。它是用 C++ 编写的，但在 C 和 C++ 项目中使用，在嵌入式测试中经常使用，但它适用于任何 C/C++ 项目。

> **为什么使用 C++ 框架测试 C？**
>
> 标准的 C 无法处理测试用例自安装问题，容易造成测试丢失[1]。

C++ 支持 C/C++ 测试，只使用有限的 C++ 语言特性。

特点：

- 指针恢复机制
- 支持 mock
- IEEE754 浮点数异常检测
- 内存泄漏检测

{{< admonition tip "提示" true >}}
`Cpputest` 仓库中有 ruby 脚本文件，能将测试的 `cpp` 文件转为 `unity` 测试用文件。
{{< /admonition >}}

| qpc | STM32 | arm-eabi-gcc | mock | RTOS |
|:---:|:-----:|:------------:|:----:|:----:|
| ✅  |  ✅  |      ✅      |  ✅  |  ✅  |

安装：

`sudo apt-get install cpputest`

项目结构，用于测试嵌入式项目时，`src`作为源码目录存放`.c`文件，`tests`中存放测试框架的`.cpp`文件，基本所有测试框架都可以按照这个方式组织。

```bash
cmake_cpputest
   ├── CMakeLists.txt
   ├── src
   │    ├── CMakeLists.txt
   │    ├── code.c     # 模板自带，添加多项式拟合函数
   │    ├── code.h     
   │    ├── crc.c      # 添加 CRC 校验函数
   │    ├── crc.h
   │    └── main.c      
   └── tests
        ├── CMakeLists.txt
        ├── codeTest.cpp   # 测试多项式拟合
        ├── crc_test.cpp   # 测试crc
        └── main.cpp
```

tests/main.cpp 测试框架 main 函数

```cpp
#include "CppUTest/CommandLineTestRunner.h"
int main(int ac, char** av)
{
    // CommandLineTestRunner 类名
    return CommandLineTestRunner::RunAllTests(ac, av);
}
```

tests/codeTest.cpp 被测函数/模块

> 需要使用 `extern "C"` 引入模块的头文件

```cpp
#include "CppUTest/TestHarness.h"
extern "C" {
  #include "code.h"
}
TEST_GROUP(AwesomeExamples)
{
};
TEST(AwesomeExamples, FirstExample)
{
  int x = test_func();
  CHECK_EQUAL(1, x);
}
```

src/code.c

```c
#include "code.h"
int test_func ()
{
 return 1;
}
```

src/code.h

```c
#ifndef __code_h__
#define __code_h__
int test_func ();
#endif
```

src/main.c

```c
#include <stdio.h>
#include "code.h"
int main(void)
{
  test_func();
  printf("hello world!\n");
  exit(0);
}
```

顶层 cmake 模板：

```c
cmake_minimum_required(VERSION 3.7)
project(cmakeCppUTestDemo)

# 使能测试功能
enable_testing()

# 设置语言标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_C_STANDARD 99)

# 禁止在源码目录修改和编译
set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)
set(CMAKE_DISABLE_SOURCE_CHANGES ON)

#-Wall: 编译后显示所有警告⚠️
add_compile_options(-Wall -Werror)

set(PROJECT fooApp)
set(APP_LIB_NAME fooAppLib)

# 子项目
add_subdirectory(src)

# 测试构建选项,源外构建 cmake -D COMPILE_TESTS ..
option(COMPILE_TESTS "Compile the tests" ON)
if(COMPILE_TESTS)
  add_subdirectory(tests)
endif(COMPILE_TESTS)
```

src/cmakelist:

```c
set(APP_LIB_SOURCE code.c)
# 添加库文件
add_library(${APP_LIB_NAME} ${APP_LIB_SOURCE})
# 添加可执行文件
add_execiteable(${PROJECT} main.c)
# 链接库
target_link_libraries(${PROJECT} ${APP_LIB_SOURCE})
```

tests/cmakelist:

```c
# 寻找安装的CppUTest
if(DEFINED ENV{CPPUTEST_HOME})
  message(STATUS "Using CppUTest home: $ENV{CPPUTEST_HOME}")
  set(CPPUTEST_INCLUDE_DIRS $ENV{CPPUTEST_HOME}/include)
  set(CPPUTEST_LIBRARIES $ENV{CPPUTEST_HOME}/lib)
  set(CPPUTEST_LDFLAGS CppUTest CppUTestExt)
else()
  find_package(PkgConfig REQUIRED)
  pkg_search_module(CPPUTEST REQUIRED cpputest>=3.8)
  message(STATUS "Found CppUTest version ${CPPUTEST_VERSION}")
endif()

# 单元测试相关变量
set(TEST_PROJECT ${PROJECT}_tests)
set(TEST_SOURCES codeTest.cpp main.cpp)

# 外部目录
# cpputest and current src
include_directories(${CPPUTEST_INCLUDE_DIRS} ../src/)
# link
link_directories(${CPPUTEST_LIBRARIES})

#
add_executable(${TEST_PROJECT} ${TEST_SOURCES})
target_link_libraries(${TEST_PROJECT} ${APP_LIB_NAME} ${CPPUTEST_LDFLAGS})

# run test once build is done
add_custom_command(TARGET ${TEST_APP_NAME} COMMAND ./${TEST_APP_NAME} POST_BUILD)
```

新增 polyfit.c polyfit.h，在 codeTest.cpp 中加入：

```cpp
#include <CppUTest/CommandLineTestRunner.h>
#include <CppUTest/TestHarness.h>
#include <CppUTestExt/MockSupport.h>

extern "C" {
  #include "code.h"
  #include "polyfit.h"
}

TEST_GROUP(AwesomeExamples)
{
};

TEST(AwesomeExamples, FirstExample)
{
  int x = test_func();
  CHECK_EQUAL(1, x);
}

TEST_GROUP(TestPolyfit)
{
    void setup() {}

    void teardown()
    {
        mock().clear();
    }
};

//----------------------------------------------------
// Testing Third Order Polynomial fitting...
//--------------------------------------------------
TEST(TestPolyfit, ThirdOrderPoly)
{
    const unsigned int order = 3;
    const unsigned int countOfElements = 5;
    const double acceptableError = 0.01;
    int result;

    // These inputs should result in the following approximate coefficients:
    //         0.5           2.5           1.0        3.0
    //    y = (0.5 * x^3) + (2.5 * x^2) + (1.0 * x) + 3.0
    double    xData[countOfElements] = {    12.0,
                                            77.8,
                                            44.1,
                                            23.6,
                                           108.2};
    double    yData[countOfElements] = {  1239.00,
                                        250668.38,
                                         47792.19,
                                          7991.13,
                                        662740.98};
    double coefficients[order + 1]; // resulting array of coefs

    // Perform the polyfit
    result = polyfit(xData,
                     yData,
                     countOfElements,
                     order,
                     coefficients);

    CHECK_EQUAL(0, result);
    DOUBLES_EQUAL(0.5, coefficients[3], acceptableError);
    DOUBLES_EQUAL(2.5, coefficients[2], acceptableError);
    DOUBLES_EQUAL(1.0, coefficients[1], acceptableError);
    DOUBLES_EQUAL(3.0, coefficients[0], acceptableError);
}

//----------------------------------------------------
// Verify incorrect result - insufficient count of xData
// and yData for the specified polyfit order.
//--------------------------------------------------
TEST(TestPolyfit, InsufficientInputData)
{
    const unsigned int order = 3;
    const unsigned int countOfElements = 2;
    int result;

    // These inputs should result in the following approximate coefficients:
    //         0.1           -3.2           -0.1        40.0
    //    y = (0.1 * x^3) + (-3.2 * x^2) + (-0.1 * x) + 40.0
    double    xData[countOfElements] = {    15.0,
                                            77.0};
    double    yData[countOfElements] = {  -344.0,
                                         26712.8};
    double coefficients[order + 1]; // resulting array of coefs

   // Perform the polyfit
   result = polyfit(xData,
                    yData,
                    countOfElements,
                    order,
                    coefficients);

   CHECK_EQUAL(-1, result);
}
```

新增 crc.c 和 crc.h 文件：

```c
const uint16_t crc_table[256] = {
    #include "crc_table.inc"
};
const uint16_t INIT = 0xffff;
uint16_t CalculateCRC(uint8_t* message, size_t length)
{
    uint16_t crc = INIT;
    uint16_t table_index;

    for (uint16_t i = 0; i < length; i++) {
        table_index = ((crc >> 8) ^ message[i]) & 0xff;
        crc = (uint16_t)((crc << 8) ^ crc_table[table_index]);
    }
    return crc;
}
```

进行了上述拟合多项式测试和 CRC 校验测试，输出情况：

![img](/UnitTesting/cpputest_output.png)

测试成功时输出最少，重点显示测试失败的位置，行数。

### 5.2 Unity

Unity 是一个用 C 语言构建的单元测试框架，专注于嵌入式领域。

[Unity github](https://github.com/ThrowTheSwitch/Unity)

|状态机 | STM32 | arm-eabi-gcc | mock | RTOS |
|:---:|:-----:|:------------:|:----:|:----:|
| ✅  |  ✅  |      ✅      |  ✅  |  ✅  |

项目结构：

```bash
.
├── CMakeLists.txt
├── include
│   └── example
│       ├── crc.h
│       └── example.h
├── src
│   ├── CMakeLists.txt
│   ├── example         # 被测模块
│   │   ├── crc.c        
│   │   ├── crc_table.inc
│   │   └── example.c
│   └── main.c
└── test
    ├── CMakeLists.txt
    ├── example
    │   ├── crc_test.c   # crc 测试模块
    │   └── main_test.c  # 测试部分主函数
    └── lib
        └── Unity        # Unity 源码
```

其中 `unity` 包含在 `tests` 目录的 `lib` 中，以第三方库的形式集成在项目中。

tests/CMakeLists.txt:

```c
# Set project name
project(Test)

# Add unity cmakes
add_subdirectory(lib/Unity)

# Add their include files 要使用完整的 Unity 功能，需要引入这三个目录
include_directories(lib/Unity/src)
include_directories(lib/Unity/extras/fixture/src)
include_directories(lib/Unity/extras/memory/src)

# Set the source of the tests
set(Sources example/crc_test.c example/main_test.c)

# Set the target executable
add_executable(Test_run ${Sources})

# Add teh target link libraries
target_link_libraries(Test_run unity)
target_link_libraries(Test_run Example_lib)
```

被测试模块 `Example_lib` 编译成库给测试模块使用。

> 使用 Unity 的 fixture 和 memory 功能时，要打开 Unity 目录下 CMakeLists.txt 文件的两个选项。
>
> ```c
> # Options to Build With Extras -------------------------------------------------
> option(UNITY_EXTENSION_FIXTURE "Compiles Unity with the \"fixture\" extension." ON)
> option(UNITY_EXTENSION_MEMORY "Compiles Unity with the \"memory\" extension." ON)
> ```

测试模块函数 tests/crc_test.c

```c
#include "unity_fixture.h"
#include "example/crc.h"

TEST_GROUP(CRC) ;

TEST_SETUP(CRC) { }

TEST_TEAR_DOWN(CRC) { }

// CRC 组测试 1
TEST(CRC, known_crc)
{
    uint8_t message1[] = {0xaa, 0xde, 0xad, 0xbe, 0xef};
    uint8_t message2[] = {0x0b, 0xb9, 0x7f, 0x4c, 0x4a, 0x32, 0xe6, 0x6b, 0x5e, 
                          0xef, 0x86, 0xb1, 0xc2, 0x4b, 0x31};
    uint16_t crc1 = CalculateCRC(message1, sizeof(message1));
    uint16_t crc2 = CalculateCRC(message2, sizeof(message2));

    TEST_ASSERT_EQUAL_UINT16(0xC149, crc1);
    TEST_ASSERT_EQUAL_UINT16(0x5BEE, crc2);
}
// CRC 组测试 2
TEST(CRC, Nonzero_output)
{
    uint8_t message[] = {0x00, 0x00, 0x00, 0x00};
    uint16_t crc = CalculateCRC(message, sizeof(message));
    TEST_ASSERT_TRUE(crc != 0);
}
// CRC 组测试 3
TEST(CRC, Long_message)
{
    uint8_t message[4096];
    uint16_t crc;
    for (int i = 0; i < sizeof(message); i++) {
        message[i] = i % 256;
    }
    crc = CalculateCRC(message, sizeof(message));
}
// 定义本组哪些测试运行
TEST_GROUP_RUNNER(CRC)
{
    RUN_TEST_CASE(CRC, known_crc);
    RUN_TEST_CASE(CRC, Nonzero_output);
    RUN_TEST_CASE(CRC, Long_message);
}
```

相比以 C++ 为基础的测试框架，Unity 看起来断言种类要更多，比如判断整数相等会分为 8，16，32 位，有无符号等。猜测由于 C++ 支持函数重载和泛型编程等特性导致。

tests/main_test.c

```c
#include "unity_fixture.h"

static void runAllTests()
{
    RUN_TEST_GROUP(CRC);
}

int main(int argc, const char * argv[]) {
    return UnityMain(argc, argv, runAllTests);
}
```

在 `build` 目录中执行：

```bash
cmake --graphviz=foo.dot ..
dot -T png foo.dot -o foo.png
```

可以生成项目依赖关系图：

![foo](/UnitTesting/foo.png)

进行了 CRC 校验测试，输出情况：

![img](/UnitTesting/Unity_output.png)

基本和 cpputest 一致。 [教学](https://vimeo.com/73660695)

### 5.3 Gtest

最常见的单元测试框架，Google 开发，功能最多，多用于 C++ 项目测试。新版本 `gtest` 和 `gmock` 已经合并。

使用源码编译安装，编译前需要打开 gmock 下 cmake 文件的编译选项。

```bash
wget https://github.com/google/googletest/archive/refs/tags/v1.14.0.tar.gz
tar -zxf v1.14.0.tar.gz
cd googletest-release-1.14.1
mkdir build && cd build
cmake ..
make
sudo make install
```

头文件安装在 `/usr/local/include/gtest/`
库文件安装在 `/usr/local/lib/`

下面是 CRC 校验测试项目结构：

```bash
.
├── CMakeLists.txt
├── build
├── src
│   ├── CMakeLists.txt
│   ├── crc.c
│   ├── crc.h
│   ├── crc_table.inc
│   └── main.c
└── tests
    ├── CMakeLists.txt
    └── crc_test.cpp
```

使用方式和前两者基本相同，`main()` 函数所在文件：

```cpp
#include <gtest/gtest.h>
extern "C" {
    #include "crc.h"
}

TEST(CRC, known_crc) {
    uint8_t message1[] = {0xaa, 0xde, 0xad, 0xbe, 0xef};
    uint8_t message2[] = {0x0b, 0xb9, 0x7f, 0x4c, 0x4a, 0x32, 0xe6, 0x6b, 0x5e, 
                          0xef, 0x86, 0xb1, 0xc2, 0x4b, 0x31};
    uint16_t crc1 = CalculateCRC(message1, sizeof(message1));
    uint16_t crc2 = CalculateCRC(message2, sizeof(message2));

    EXPECT_EQ(0xC149, crc1);
    EXPECT_EQ(0x5BEE, crc2);
}

TEST(CRC, Nonzero_output) {
    uint8_t message[] = {0x00, 0x00, 0x00, 0x00};
    uint16_t crc = CalculateCRC(message, sizeof(message));
    EXPECT_TRUE(crc != 0);
}

TEST(CRC, Long_message)
{
    uint8_t message[4096];
    uint16_t crc;
    for (int i = 0; i < int(sizeof(message)); i++) {
        message[i] = i % 256;
    }
    crc = CalculateCRC(message, sizeof(message));
    EXPECT_EQ(0x1f5, crc);  
    //EXPECT_EQ(0x1fb5, crc);
}

int main(int argc, char **argv)
{
    printf("Running main() from %s\n", __FILE__);
    testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS(); 
}
```

输出：

![img](/UnitTesting/gtest_output.png)

### 5.4 cmocka

cmocka, C 单元测试框架，支持 mock。只依赖 C 标准库，由 C 语言开发。

API:

![cmocka](/UnitTesting/cmocka.png)

{{< admonition warning "注意" true >}}
Misra 规则 Rule21.4 必然无法在单元测试中被满足，所有框架的底层都依赖头文件 `setjmp.h`
{{< /admonition >}}

项目模板：

[cmocka 单元测试模板](https://github.com/NikLeberg/cmake-cmocka-tdd-template)

可以以此为基准构建测试项目：

command | action | &nbsp;
---|---|---
`make test_cppcheck` | check source code with cppcheck | &nbsp;
`make` | compile source code | executable can be found under [build/src/](build/src/)
`cmake ..` | generate buildsystem | &nbsp;
`make doc` | generate documentation | to view open [build/doc/html/index.html](build/doc/html/index.html)
`make clean` | remove compiled files | &nbsp;
`make tests` | run all tests  | cmocka & cppcheck
`make test_cmocka` | run cmocka tests | &nbsp;

项目框架：

```bash
.
├── build
├── CMakeLists.txt
├── doc
│   ├── CMakeLists.txt
│   └── Doxyfile.in
├── lib
│   └── CMakeLists.txt
├── src
│   ├── CMakeLists.txt
│   ├── main.c
│   └── mylib
│       ├── CMakeLists.txt
│       ├── crc.c
│       ├── crc.h
│       ├── crc_table.inc
│       ├── mylib.c
│       └── mylib.h
└── test
    ├── CMakeLists.txt
    ├── mylib.c
    └── test.h
```

测试函数使用相对前三者较复杂，需要手动将测试函数添加到主函数的编组。

```c
/***优先包含这几项***/
#include <stdarg.h>
#include <stddef.h>
#include <setjmp.h>   // 可能违反 Misra Rule 21.4
#include <stdint.h>   
/***否则无法通过编译***/
#include <cmocka.h>

//...

static void says_hello(void **state) {
    (void)state;
    char str[10];
    mylib_sayHello(str);
    assert_string_equal("Hello", str);
}

static void crc_kownc(void **state) {
    (void)state;
    uint8_t message1[] = {0xaa, 0xde, 0xad, 0xbe, 0xef};
    uint8_t message2[] = {0x0b, 0xb9, 0x7f, 0x4c, 0x4a, 0x32, 0xe6, 0x6b, 0x5e, 
                          0xef, 0x86, 0xb1, 0xc2, 0x4b, 0x31};
    uint16_t crc1 = CalculateCRC(message1, sizeof(message1));
    uint16_t crc2 = CalculateCRC(message2, sizeof(message2));

    assert_uint_equal(0xC149, crc1);
    assert_uint_equal(0x5BEE, crc2);
}

static void Nonzero_output(void **state) {
    (void)state;
    uint8_t message[] = {0x00, 0x00, 0x00, 0x00};
    uint16_t crc = CalculateCRC(message, sizeof(message));
    assert_true(crc != 0);
}

static void Long_message(void **state) {
    (void)state;
    uint8_t message[4096];
    uint16_t crc;
    for (int i = 0; i < (int)(sizeof(message)); i++) {
        message[i] = i % 256;
    }
    crc = CalculateCRC(message, sizeof(message));
    assert_uint_equal(0x1f5, crc);  
    //TEST_ASSERT_EQUAL_UINT16(0x1fb5, crc);
}

int main(void) {
    const struct CMUnitTest tests_mylib[] = {
        cmocka_unit_test(says_hello),
        cmocka_unit_test(crc_kownc),
        cmocka_unit_test(Nonzero_output),
        cmocka_unit_test(Long_message),
    };
    return cmocka_run_group_tests(tests_mylib, NULL, NULL);
}
```

输出内容和 goolgetest 类似，两者都是 google 开发的：

![cmocka output](/UnitTesting/cmocka_output.png)

### 5.5 CUnit

其他框架主函数部分基本不用改动，相比之下 `CUnit` 使用不便捷，虽然能以列表的方式输出结果。

`main()`函数部分：

```c
#include ...

int main()
{
    CU_pSuite pSuite = NULL;
   
    //初始化 CUnit 注册表
    if(CUE_SUCCESS != CU_initialize_registry())
        return CU_get_error();
   
    //向 CUnit 注册表添加单元
    pSuite = CU_add_suite("Suite_1", init_suite1, clean_suite1);
    if(NULL == pSuite) {
        CU_cleanup_registry();  //清空注册表，释放内存
        return CU_get_error();  //返回错误
    }
   
    //向单元添加测试用例
    if((NULL == CU_add_test(pSuite, "test of fprintf()", testFPRINTF)) ||
        (NULL == CU_add_test(pSuite, "test of fread()", testFREAD))) {
        CU_cleanup_registry();
        return CU_get_error();
    }
   
    //使用 CUnit Basic 接口运行所有测试单元
    CU_basic_set_mode(CU_BRM_VERBOSE);
    CU_basic_run_tests();
   
    CU_cleanup_registry();
    return CU_get_error();
}

```

### 5.6 Criterion

特点：

- 自动注册测试
- 不需要声明 main 函数
- 测试过程隔离
- hook 机制跟踪统计

使用便捷性好，但是不支持 mock 是主要问题。

### 5.7 Mock

全网关于单元测试的内容远大于 mock 相关内容。

#### 5.7.1 Gmock

用 gmock 模拟 C 函数接口需要将其重写到一个类中，并且写成虚函数[2]。

![img](/UnitTesting/serialport_def.png)

![img](/UnitTesting/serialport_mock.png)

官方：[Mocking Free Functions](https://gitlab-public.fz-juelich.de/j.coenen/frida/-/blob/ad2120277a2871efd21990c28e787360a513e861/pub/3rdparty/yaml-cpp/test/gtest-1.8.0/googlemock/docs/v1_6/CookBook.md)

#### 5.7.2 CppUTest

![img](/UnitTesting/cppumock.png)

#### 5.7.3 CMock

和 Unity 同一作者写的 mock 框架。但案例很少。

#### 5.7.4 cmocka

cmocka 相关资料很少
[cmocka](https://lwn.net/Articles/558106/)

#### 5.7.5 其他独立的 mock 库

1. [fff](https://github.com/meekrosoft/fff)
2. [cpp-stub](https://github.com/coolxv/cpp-stub)

**参考回答：**

1. [嵌入式设备测试问题](https://stackoverflow.com/questions/60552301/google-test-for-embedded-systems/65238678#65238678)

    描述了一个嵌入式设备测试问题，解决方案是使用一大一小两个框架，部署前主要在 PC 上进行测试，使用 Gtest 或者 CppUTest，实体设备使用 minunit(只有一个头文件，300 多行)。

2. [Unit Testing Frameworks for C: Comparison](https://stackoverflow.com/questions/1468110/unit-testing-frameworks-for-c-comparison/18972102#18972102)

## 6. 测试用例设计

### 6.1 测试数据

1. 基于需求的测试
2. 正面 / 负面测试
3. 边界值分析

   ![img](/UnitTesting/边界.png)

4. 决策表
5. 等效类划分
6. 基于状态的测试
7. 用户文档测试

### 6.2 测试用例表

![img](/UnitTesting/test_case_table.jpg)

## 7. 其他

### 7. 1 qemu

官方的 qemu 不支持所有的 STM32 模拟功能，类似的开源项目只针对特定型号进行过二次开发。

![img](/UnitTesting/qemu.png)

官方：
[STMicroelectronics STM32 boards (netduino2, netduinoplus2, stm32vldiscovery) — QEMU documentation](https://www.qemu.org/docs/master/system/arm/stm32.html)

非官方：

[GitHub - beckus/qemu_stm32: QEMU with an STM32 microcontroller implementation](https://github.com/beckus/qemu_stm32)

### 7. 2 测试用例生成

1. allpairspy

   参数组合测试工具

   ```python
   from allpairspy import AllPairs

   parameters = [
       ["Brand X", "Brand Y"],
       ["98", "NT", "2000", "XP"],
       ["Internal", "Modem"],
       ["Salaried", "Hourly", "Part-Time", "Contr."],
       [6, 10, 15, 30, 60],
   ]

   print("PAIRWISE:")
   for i, pairs in enumerate(AllPairs(parameters)):
       print("{:2d}: {}".format(i, pairs))
   ```

   [1]. [allpairspy](https://pypi.org/project/allpairspy/)

2. KLEE

   基于苹果 LLVM 的开源测试样例检测工具。需要 clang 编译环境，可运行在 docker 中。

   ![img](/UnitTesting/KLEE.png)

   [1]. [KLEE\_为复杂系统程序自动生成高覆盖率的测试](https://zhuanlan.zhihu.com/p/350222671)

   [2]. [KLEE Symbolic Execution Engine](https://klee.github.io/)

3. [生成 ASN1 测试用例，模拟恶意数据](https://github.com/n7space/asn1scc.Fuzzer)

4. 商业工具

   - Parasoft C/C++test
   - Tessy

5. Monkey 测试

    [Monkey](https://developer.android.com/studio/test/monkey?hl=zh-cn)是一个 Android 设备（模拟器或真实设备）上的一个程序，可以产生大量随机的用户输入事件，如点击、触摸、手势等。因此 Monkey 可用于 UI 上的压力测试。

### 7. 3 主要参考书

[1]. [测试驱动的嵌入式 C 语言开发](https://book.douban.com/subject/10455879/)

[2]. [软件单元测试入门与实践](https://www.armbbs.cn/forum.php?mod=viewthread&tid=95201)

[3]. [C 现代编程](https://book.douban.com/subject/26756137/)

[4]. [嵌入式软件测试](https://book.douban.com/subject/3149471/)

[5]. [代码修改的艺术](https://book.douban.com/subject/2248759/)

### 7.4 参考

1. [从零搭建一个 c/c++ 工程 - 将 gtest 引入到项目中](https://www.bilibili.com/video/BV1AX4y1J7dh/?spm_id_from=333.999.0.0&vd_source=d669a99eaef48bca1935ad7a52416701)
2. [unit test mocking](https://interrupt.memfault.com/blog/unit-test-mocking)
3. [详解圈复杂度](https://cloud.tencent.com/developer/article/1900402)

## todo

- [ ] mock 部分

