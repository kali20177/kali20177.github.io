# Opencv Cuda 版本编译流程


## 1. 说明

项目需要对比 opencv 和 halcon 在某些算法执行上的速度。记录一下编译 opencv cuda 版本的流程。

## 2. 工具

> opencv cuda 版本的编译需要各种工具版本相互配合。以下组合是在我在 windows 11 上编译成功使用的组合。(OpenCV 4.4 替换成新版本的 3.4 也可)

* CMake 3.18
* CUDA Toolkit 11.0
* Visual Studio 2022
* OpenCV 4.4 & OpenCV_contrib 4.4

## 3. 编译流程

### 3.1 CUDA

安装 CUDA11 和 对应的 CUDNN，并配置环境变量。

### 3.2 CMake

将 OpenCV_contrib 4.4 和 OpenCV 4.4 分别解压。打开 CMake-gui 设置源码目录和目标目录。

![img](/opencv_cuda/opencv_cuda_cmake_src.png)

点击“Configure”按钮。生成选项后，勾选/取消勾选相关内容：

* 打开 WITH_TBB
* 勾选和 CUDA 相关选项
* 取消选择 java 和 python 的编译选项 (个人不需要)
* 勾选 nonfree 的编译选项
* 取消 TEST 相关选项加快编译速度
* 勾选 BUILD_opencv_world 最终只生成一个动态链接库方便使用
* 在 OPENCV_EXTRA_MODULES_PATH 中填入 OpenCV_contrib 解压文件夹 modules 的路径

  ![img](/opencv_cuda/opencv_contrib_extra.png)

* 取消选择 OPENCV_GENERATE_SETUPVARS

再次点击 configure 按钮，由于网络问题会出现无法下载的错误，可以取消勾选 face 模块，然后再次点击 configure 按钮。使用 OpenCV4.5 时这一步多出很多下载错误，取消勾选 face 还有很多，所以换回了 4.4 版本。然后勾选 CUDA_FAST_MATH 点 configure 按钮。

> 要保证生成 CMakeLists.txt 过程中不出现红色警告，不然后续的编译过程会出现错误。

无错误后点击 Generate 按钮，生成 Opencv.sln 文件。

## 3.3 VS2022

打开生成的 OpenCV.sln 文件，找到项目 ALL_BUILD，分别在 Debug 和 Release 下右击生成。编译大概需要六个小时。可以使用批生成功能。

![img](/opencv_cuda/batch_build.png)

ALL_BUILD 完成后，找到项目 INSTALL 右击生成。最终会在当前 build 目录下生成一个 install 目录，这就是我们需要的可执行文件和库，最后添加到环境变量中测试效果。

## 4. 效果

将 install 目录中的 bin 目录添加到环境变量中，推荐使用环境变量管理工具 rapid environment editor。

![img](/opencv_cuda/rapid_environment_editor.png)

bin 文件在 install/x64/vc16 中：

![img](/opencv_cuda/opencv_path.png)

打开项目属性 VC++ 目录，分别设置

包含目录：

![img](/opencv_cuda/opencv_cuda_include.png)

库目录：

![img](/opencv_cuda/opencv_cuda_lib.png)

在 链接器 --> 输入 --> 附加依赖项 中，分别在 debug 和 release 模式下添加 opencv_world3415d.lib、opencv_world3415.lib
至此配置完毕。

添加头文件，即可开始使用 cuda.

```cpp
#include <opencv2/core/cuda.hpp> 

int main()
{
	cuda::printCudaDeviceInfo(cuda::getDevice());
	int count = cuda::getCudaEnabledDeviceCount();
	printf("GPU Device Count : %d\n", count);
	printf("OpenCV version: %s\n", CV_VERSION);
}
```

## 参考

1. [Win10 使用 VS2019 从源码编译 OpenCV 4.4 + CUDA 11.0 + Cudnn 8.0 + python3](https://www.jianshu.com/p/aa8455fcc672)
2. [OpenCV + CUDA 开发 - 编译与环境配置](https://www.bilibili.com/video/av71643385)

