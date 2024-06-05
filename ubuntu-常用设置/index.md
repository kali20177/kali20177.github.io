# Ubuntu 相关常用问题


## 目录名称

gnome 桌面改回英文目录名：

```bash
export LANG=en_US
xdg-user-dirs-gtk-update
export LANG=zh_CN
```

## fonts

Linux 上字体一般被存放在下面的文件夹（可通过 `cat /etc/fonts/fonts.conf` 查看）：

- `/usr/share/fonts/`
- `/usr/local/share/fonts/`
- `~/.fonts/` 用户

安装 TrueType 字体

```bash
sudo apt install ttf-mscorefonts-installer
```

显示当前安装的字体：

```bash
fc-list
```

如果需要手动安装字体，需要将解压后的字体文件或文件夹移动到上述的三个位置之一，然后刷新字体缓存：

```bash
fc-cache -f -v
```

## 编译工具链

### gcc 14

下载和解压 gcc14.1：

```bash
wget https://mirrors.tuna.tsinghua.edu.cn/gnu/gcc/gcc-14.1.0/gcc-14.1.0.tar.gz
tar -xzvf gcc-14.1.0.tar.gz
cd gcc-14.1.0/
```

编译，官方 [文档](https://gcc.gnu.org/install/configure.html) 中解释了各 configure 参数：

```bash
# 安装依赖
sudo apt install libgmp-dev libmpc-dev libmpfr-dev libmpfrc++-dev libisl-dev texinfo
# 如果执行了上一步依赖安装可以不用执行此脚本
./contrib/download_prerequisites

mkdir build && cd build
# --program-suffix 会给安装的 gcc 程序添加后缀
../configure --prefix=/opt/gcc-14.1.0 --enable-checking=release --enable-languages=c,c++,go --disable-multilib --enable-bootstrap --enable-threads=posix --program-suffix=-14

make -j$(nproc)
sudo make install
```

环境变量设置：

```bash
# gcc-14.1 ENV
export GCC14_HOME=/opt/gcc-14.1.0
export PATH=$GCC14_HOME/bin:$PATH
export LD_LIBRARY_PATH=$GCC14_HOME/lib64:$LD_LIBRARY_PATH
```

Gcc 14 优化了静态分析特性，可以通过编译选项 `-fanalyzer` 检查死代码，可视化 buffer overflow 等。例如对下面的代码：

```c
void test (void)
{
  char buf[10];
  strcpy (buf, "hello");
  strcat (buf, " world!");
}
```

编译将输出：

```bash
<source>:7:3: note: write of 3 bytes to beyond the end of 'buf'
    7 |   strcat (buf, " world!");
      |   ^~~~~~~~~~~~~~~~~~~~~~~
<source>:7:3: note: valid subscripts for 'buf' are '[0]' to '[9]'

                             ┌────┬────┬────┬────┬────┐┌─────┬─────┬─────┐
                             │[0] │[1] │[2] │[3] │[4] ││ [5] │ [6] │ [7] │
                             ├────┼────┼────┼────┼────┤├─────┼─────┼─────┤
                             │' ' │'w' │'o' │'r' │'l' ││ 'd' │ '!' │ NUL │
                             ├────┴────┴────┴────┴────┴┴─────┴─────┴─────┤
                             │     string literal (type: 'char[8]')      │
                             └───────────────────────────────────────────┘
                               │    │    │    │    │      │     │     │
                               │    │    │    │    │      │     │     │
                               v    v    v    v    v      v     v     v
  ┌─────┬────────────────────┬────┬──────────────┬────┐┌─────────────────┐
  │ [0] │        ...         │[5] │     ...      │[9] ││                 │
  ├─────┼────┬────┬────┬────┬┼────┼──────────────┴────┘│                 │
  │ 'h' │'e' │'l' │'l' │'o' ││NUL │                    │after valid range│
  ├─────┴────┴────┴────┴────┴┴────┴───────────────────┐│                 │
  │             'buf' (type: 'char[10]')              ││                 │
  └───────────────────────────────────────────────────┘└─────────────────┘
  ├─────────────────────────┬─────────────────────────┤├────────┬────────┤
                            │                                   │
                  ╭─────────┴────────╮              ╭───────────┴──────────╮
                  │capacity: 10 bytes│              │⚠️  overflow of 3 bytes│
                  ╰──────────────────╯              ╰──────────────────────╯
...
```

### CMake

没有 cmake 环境：

```bash
./bootstrap --qt-gui
make && sudo make install
```

使用 CMake 构建：

```bash
cmake -D BUILD_QtDialog=ON ..
make && sudo make install
```

### gdb 14.2

注意：目前不能使用 gcc-14 编译。

```bash
sudo apt install libreadline-dev libncurses*
mkdir build && cd build
../configure --prefix=/opt/gdb-14.2 --enable-targets=all --with-python --enable-lto --enable-vtable-verify
make -j6 && sudo make install
```

## 优化设置

点击程序坞图标，程序最小化：

```bash
gsettings set org.gnome.shell.extensions.dash-to-dock click-action 'minimize'
```

关闭动效

```bash
gsettings set org.gnome.desktop.interface enable-animations false
```

隐藏桌面主目录和回收站图标：

```bash
gsettings set org.gnome.shell.extensions.ding show-home false
gsettings set org.gnome.shell.extensions.ding show-trash false
```

音视频编解码器：

```bash
sudo apt-get install ubuntu-restricted-extras
```

音频控制：

```bash
sudo apt install pulseaudio pavucontrol
```

gnome 插件和优化选项：

```bash
sudo apt install gnome-tweaks
sudo apt install gnome-shell-extensions
sudo apt install chrome-gnome-shell
```

切换 Wayland 和 X11，在位置 `/etc/gdm3/custom.conf` 根据注释操作。

## fcitx 5

安装 fcitx5 本体：

```bash
sudo apt install fcitx5 fcitx5-chinese-addons fcitx5-frontend-gtk4 fcitx5-frontend-gtk3 fcitx5-frontend-gtk2 fcitx5-frontend-qt5
```

安装中文词库，[下载页](https://github.com/felixonmars/fcitx5-pinyin-zhwiki/releases)：

```bash
wget https://github.com/felixonmars/fcitx5-pinyin-zhwiki/releases/download/0.2.4/zhwiki-20240426.dict
mkdir -p ~/.local/share/fcitx5/pinyin/dictionaries/
mv zhwiki-20220416.dict ~/.local/share/fcitx5/pinyin/dictionaries/
```

设为默认输入法，终端输入 `im-config`，按照提示设置。并添加环境变量：

```bash
export XMODIFIERS=@im=fcitx
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
```

ubuntu 20.04 下 fcitx5 暂时不能图形化配置，安装命令：

```bash
sudo apt install fcitx5 fcitx5-chinese-addons fcitx5-frontend-gtk3 fcitx5-frontend-gtk2 fcitx5-frontend-qt5
```

同样在 im-config 中设置系统输入法框架为 fcitx5，添加到开机启动项。新建文件 `~/.config/fcitx5/profile`：

```text
[Groups/0]
# Group Name
Name=Default
# Layout
Default Layout=us
# Default Input Method
DefaultIM=pinyin

[Groups/0/Items/0]
# Name
Name=keyboard-us
# Layout
Layout=

[Groups/0/Items/1]
# Name
Name=pinyin
# Layout
Layout=

[GroupOrder]
0=Default
```

修改 fcitx5 的配置文件时注意：**先退出 fcitx5 再修改配置项，因为 fcitx5 进程退出后会覆写所有配置文件**。

在文件 `~/.config/fcitx5/conf/pinyin.conf` 中通过修改 PageSize 值调整候选字个数。重启后即可正常使用。

主题配置参见 github 项目 [Fcitx5-Material-Color](https://github.com/hosxy/Fcitx5-Material-Color)

## neovim

安装剪贴板依赖：

```bash
sudo apt install xclip
```

安装本体：

```bash
curl -LO https://github.com/neovim/neovim/releases/latest/download/nvim-linux64.tar.gz
sudo rm -rf /opt/nvim
sudo tar -C /opt -xzf nvim-linux64.tar.gz

export PATH="$PATH:/opt/nvim-linux64/bin"
```

## 软件版本管理

对于使用 apt 安装的多个版本的包，可以使用 update-alternatives 命令管理软链接，进行版本切换。例如 editor 命令一般有以下的替代项：

```bash
± update-alternatives --config editor
有 5 个候选项可用于替换 editor (提供 /usr/bin/editor)。

  选择       路径              优先级  状态
------------------------------------------------------------
* 0            /bin/nano            40        自动模式
  1            /bin/ed             -100       手动模式
  2            /bin/nano            40        手动模式
  3            /usr/bin/code        0         手动模式
  4            /usr/bin/vim.basic   30        手动模式
  5            /usr/bin/vim.tiny    15        手动模式
```

使用 list 就只会列出当前备选项：

```bash
± update-alternatives --list editor
```

查看 editor 的链接位置可以发现他指向了 `/etc/alternatives/editor`，这个位置的软链再次指向 nano 的位置。

以切换 gcc 版本为例，如果计算机中已经安装了 gcc-11 和 gcc-12，此时 gcc 命令默认指向 gcc-11，通过下面的命令添加新的软链：

```bash
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 10
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 20
```

语法：`--install link name path priority`

参数：
- `--install`, Add a group of alternatives to the system
- `/usr/bin/gcc`, the generic name for the master link
- gcc, name is the name of its symlink in the alternatives directory
- path 添加命令的实际路径
- priority 优先级

现在就给 gcc 添加了两个候选项，可通过 display 和 config 查看和设置：

```bash
update-alternatives --display gcc
update-alternatives --config gcc
```

可以使用 GUI 程序 galternatives 管理替代项：

```bash
sudo apt install galternatives
```

## Virtualbox

共享文件夹提示无权限访问，因为 `/media` 下挂载的文件夹用户是 root, 应把当前用户添加到 vboxsf 组中，执行命令后重启。

```bash
sudo usermod -aG vboxsf $(whoami)
```

## multipass

Ubuntu 官方提供的 multipass 程序能够更方便得管理 ubuntu 虚拟机，尤其在 Apple M 系列芯片的机型。安装程序后，使用下面的命令创建新的虚拟机：

```bash
multipass launch -n ubuntu24 --cpus 4 --disk 32G --memory 4G
```

操作方式和 docker 类似：

```bash
multipass shell ubuntu24    # 交互方式进入虚拟机
multipass list              # 列出当前实例
multipass delete ubuntu24   # 删除实例
multipass recover ubuntu24  # 恢复实例
multipass purge             # 永久删除实例
multipass find              # 查询支持的镜像列表
```

> 对于 arm 架构的 ubuntu，需要更换 [Ubuntu Ports](https://mirrors.ustc.edu.cn/help/ubuntu-ports.html#__tabbed_2_2) 源。

## WSL 默认系统

如果安装过 Docker Desktop，则可能会改变 WSL 的默认系统设置，导致 VSCode 选择“连接到 WSL”选项时，会连接到 docker-desktop 导致失败。下面的选项修改默认设置：

```bash
> wsl -l -v
  NAME                   STATE           VERSION
* docker-desktop         Stopped         2
  docker-desktop-data    Stopped         2
  Arch                   Stopped         2
  Ubuntu-24.04           Running         2
> wslconfig /setdefault Ubuntu-24.04
```

## 参考

1. [How to install and manage fonts on Linux](https://linuxconfig.org/how-to-install-and-manage-fonts-on-linux)
2. [GCC-13.2.0 编译安装](https://cuterwrite.top/p/gcc-13-source-install/)
3. [Improvements to static analysis in the GCC 14 compiler](https://developers.redhat.com/articles/2024/04/03/improvements-static-analysis-gcc-14-compiler#solving_the_halting_problem_)
4. [在 Ubuntu 安装配置 Fcitx 5 中文输入法](https://muzing.top/posts/3fc249cf/)
5. [如何现在就在 Ubuntu 20.04 用上 Fcitx 5](https://plumz.me/archives/11740/)
6. [在 Ubuntu Linux 上安装 Fcitx5 中文输入法](https://www.qinxu.net/linux/2021/0930636/)
7. [我也！用上 fcitx5 了](https://iovxw.net/p/fcitx5/)
8. [The update-alternatives Command in Linux](https://www.baeldung.com/linux/update-alternatives-command)
9. [VirtualBox: linux 没有权限访问共享文件夹的问题](https://www.cnblogs.com/yongdaimi/p/13424855.html)
10. [虚拟机管理工具 multipass 使用笔记](https://blog.yasking.org/a/notes-on-multipass.html)
11. [WSL2 切换默认的 Linux 子系统](https://blog.csdn.net/weixin_43425561/article/details/132478445)

