# Qt 相关


## Mac 下 GUI 程序不显示

编译示例图形界面程序后，程序坞出现图标但是不显示窗体。需要在项目的 .pro 文件添加：

```
CONFIG += sdk_no_version_check
QMAKE_MACOSX_DEPLOYMENT_TARGET = 10.15
```

清理当前构建后重新编译即可。

## Linux 下 QM 无法启动

ubuntu 下安装 QP/C 框架后，qm 程序无法启动。需要安装依赖项：

```
sudo apt install qtwayland5 '^libxcb.*-dev' libx11-xcb-dev libglu1-mesa-dev libxrender-dev libxi-dev libxkbcommon-dev libxkbcommon-x11-dev
```

## 参考

1. [mac qt问题记录](https://www.cnblogs.com/liubenben/p/17947031)
2. [qt.qpa.plugin: Could not load the Qt platform plugin "xcb" in "" even though it was found.](https://forum.qt.io/topic/127696/qt-qpa-plugin-could-not-load-the-qt-platform-plugin-xcb-in-even-though-it-was-found)
