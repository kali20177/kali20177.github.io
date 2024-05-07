# Matplotlib Cpp C++ 中使用 Python 绘图模块的方法


## 1. 说明

笔者项目中使用 halcon 图像处理软件导出的 C++ 代码组织工程。为测试 halcon 部分算法的效果，需要对图像序列进行处理，且需要查看数据的形状。在 C++ 中不依赖图形框架直接绘图可以使用 matplotlib-cpp 来实现。在此记录一种最快的方法。

## 2. 过程

### 2.1 安装必要的包

首先需要 python 环境，并且安装 matplotlib 和 numpy。
下一步安装 vcpkg 作为 C++ 的包管理器。

```bash
git clone https://github.com/microsoft/vcpkg
cd vcpkg
.\bootstrap-vcpkg.bat
.\vcpkg integrate install
```

执行完毕后，vcpkg 就被集成到了 VS2022 中。

### 2.2 安装 matplotlib-cpp

```bash
.\vcpkg install matplotlib-cpp
.\vcpkg install matplotlib-cpp:x64-windows
```

没有报错后可进行下一步。

### 2.3 配置 VS2022

1. 添加附加包含目录

    打开项目 --> 属性。
    在 C++ --> 常规 中添加本地 python 的 include 文件夹路径和 numpy 的 include 路径。
    ![img](/matplotlib-cpp/python_include.png)

2. 添加附加库目录

    在 链接器 --> 常规 中添加 python libs 路径。
    ![img](/matplotlib-cpp/linker.png)

3. input

    在 链接器 --> 输入 中添加 python 的 lib 文件。
    ![img](/matplotlib-cpp/input.png)

4. 修改 matplotcpp.h 文件

    打开 matplotlib-cpp.h 文件，在头文件中添加 `include <string>`。不然会报"命名空间 std 没有成员 stod"的错误。

    > 关于这个错误，我查看项目的 github 仓库，源文件中已经添加了 string 头文件，但通过 vcpkg 安装的版本仍需添加。

    注释掉 340 行左右的两个 template 定义。
    
    ![img](/matplotlib-cpp/template.png)

## 3. 运行效果

计算图像序列的锐度值后绘制图像，观察尖峰性。使用时只需要添加头文件和命名空间，

```cpp
#include "matplotlibcpp.h"
namespace plt = matplotlibcpp;
```

把想要查看的向量传入 plot 即可。结果如下：

![img](/matplotlib-cpp/res.png)

## 参考

1. [VisualStudio2019 c++ 安装 matplotlib-cpp](https://zhuanlan.zhihu.com/p/310073847)
2. [【C++】11 Visual Studio 2019 C++ 安装 matplotlib-cpp 绘图](https://blog.csdn.net/weixin_43012724/article/details/124051588)

