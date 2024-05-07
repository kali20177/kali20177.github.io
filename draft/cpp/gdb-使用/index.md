# Gdb 使用


## Core dump

Linux 下默认关闭该功能，开启命令：

```shell
ulimit -c unlimited
```

使用 GDB 进行调试时，可以通过两种方式来进行调试：一种是直接调试可执行文件，另一种是调试 core 文件。以下是二者的区别：

1. 调试可执行文件：在 debug 模式下编译的可执行文件中包含了调试信息，包括了源代码的行号、变量名等信息。通过调试可执行文件，你可以在 GDB 中查看源代码、设置断点、单步执行等操作。这种方式适用于在程序运行过程中进行实时的调试和跟踪。

2. 调试 core 文件：当程序发生崩溃并生成了 core dump 文件时，你可以使用 GDB 加载这个 core 文件进行调试。core 文件包含了程序崩溃时的内存快照，通过加载 core 文件，你可以在 GDB 中查看程序崩溃时的内存状态、变量值、函数调用栈等信息。这种方式适用于分析程序崩溃的原因。

### 发生原因

1. 访问越界：字符串结束符、空指针、野指针、下标越界
2. 多线程
3. 堆栈溢出：太大的局部变量，无限嵌套、递归或调用函数

|          Signal |            Action |                      Comment                |
|:---------------:|:-----------------:|:-------------------------------------------:|
|         SIGQUIT |             Core  |               Quit   from keyboard          |
|          SIGILL |             Core  |               Illegal   Instruction         |
|         SIGABRT |             Core  |             Abort   signal from abort       |
|         SIGSEGV |             Core  |            Invalid   memory reference       |
|         SIGTRAP |             Core  |              Trace/breakpoint   trap        |

## 技巧

打印时显示数组下标：

```shell
set print array-indexes on
```

打印结构体数组成员指针指向的数组，例如：

```c
typedef struct
{
    uint32_t offset;
    uint32_t len;
    void *buf;
} Wp;

Wp wp[20];
...
wp[0].buf = (void *)array;
```

16 进制打印 `wp[0].buf` 指向的 array

```shell
p /x *(uint8_t *)(wp[0].buf)@len
```

## VSCode 打印 STL

gdb 调试 C++ 程序时，默认不能直接显示 STL 容器的值。[微软文档](https://code.visualstudio.com/docs/cpp/config-mingw) 的配置文件 `launch.json` 中：

```json
{
  "configurations": [
    {
      "name": "(gdb) 启动",
      "type": "cppdbg",
      "request": "launch",
      "program": "${workspaceFolder}/build/demo",
      "args": [],
      "stopAtEntry": false,
      "cwd": "${workspaceFolder}/build/",
      "environment": [],
      "externalConsole": false,
      "MIMode": "gdb",
      "miDebuggerPath": "gdb",
      "setupCommands": [
          {
              "description": "为 gdb 启用整齐打印",
              "text": "-enable-pretty-printing",
              "ignoreFailures": true
          },
          {
              "description": "将反汇编风格设置为 Intel",
              "text": "-gdb-set disassembly-flavor intel",
              "ignoreFailures": true
          }
      ]
  }
  ],
  "version": "2.0.0"
}
```

`"text": "-enable-pretty-printing"` 能够打开 STL 的查看。注意使用时修改参数： `program`, `cwd`，必要时填写编译器路径。

> windows 上使用 msys2 有效，使用 w64devkit 无效。

## 反向调试

要求 gdb 版本大于 7.0 并且支持反向调试的平台。在 windows 下使用 `w64devkit` 的 gcc 会出现以下提示：

```shell
Process record: the current architecture doesn't support record function.
```

在 linux 上此功能正常。

## 参考

1. [linxu core dump](https://www.cnblogs.com/hazir/p/linxu_core_dump.html)
2. [gdb print 设置](https://zhuanlan.zhihu.com/p/594423089)
3. [一文读懂 | coredump 文件是如何生成的](https://cloud.tencent.com/developer/article/1860631)
4. [GDB 反向调试（Reverse Debugging）](https://www.cnblogs.com/xuxm2007/p/3244709.html)
5. [GDB 调试工具高级用法](https://www.cnblogs.com/wintrysec/p/10485016.html#yNN8cd4J)

