# Linux 下名称缩写和解释


## 1. tty

TTY，teletype 的缩写。
早期终端是一种电传打字机，并且最早终端和控制台不是一种东西。
数控设备的控制设备一般叫控制台，终端是通过串口连接上的，不是计算机本身的设备。能显示系统运行状态的叫控制台，其他叫终端。

![tty](/Linux_abbr/tty.png)

但是发展到现在基本已经不区分二者的概念。终端渐渐发展成了一个软件概念。

![pty](/Linux_abbr/pty.png)

现代计算机的 terminal 程序是运行在用户态的仿真程序，称为伪终端 (pty)。

## 2. console

## 3. coredump

当程序运行的过程中异常终止或崩溃，操作系统会将程序当时的内存状态记录下来，保存在一个文件中，这种过程被称为 Core Dump（核心转储）。
> 在半导体作为电脑内存材料之前，电脑内存使用的是磁芯内存（Magnetic Core Memory），Core Dump 中的 Core 沿用了磁芯内存的 Core 表达。

![Magnetic Core Memory](/gdb/Magnetic_Core_Memory.png)

> Dump 指的是拷贝一种存储介质中的部分内容到另一个存储介质，或者将内容打印、显示或者其它输出设备。dump 出来的内容是格式化的，可以使用一些工具来解析它。

## 4. usr

