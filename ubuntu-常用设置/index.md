# Ubuntu 常用设置


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

## gcc 13

下载和解压 gcc13.2：

```bash
wget https://mirror.koddos.net/gcc/releases/gcc-13.2.0/gcc-13.2.0.tar.gz
tar -xzvf gcc-13.2.0.tar.gz
cd gcc-13.2.0
```

编译：

```bash
sudo apt install libgmp-dev libmpc-dev libmpfr-dev libmpfrc++-dev libisl-dev
./contrib/download_prerequisites
mkdir build && cd build

../configure --prefix=/opt/gcc-13.2.0 --with-pkgversion='glibc gcc V13.2.0' --enable-checking=release --enable-languages=c,c++ --disable-multilib --enable-bootstrap --enable-threads=posix --with-system-zlib 

make -j$(nproc)
make install
```

环境变量设置：

```bash
# gcc-13.2 ENV
export GCC13_HOME=/opt/gcc-13.2.0
export PATH=$GCC13_HOME/bin:$PATH
export LD_LIBRARY_PATH=$GCC13_HOME/lib64:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=$GCC13_HOME/lib:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=$GCC13_HOME/libexec:$LD_LIBRARY_PATH
export CPATH=$GCC13_HOME/include:$CPATH
export INCLUDE=$GCC13_HOME/include:$CPATH
export CC=$GCC13_HOME/bin/gcc
export CXX=$GCC13_HOME/bin/g++
```

## 优化设置

点击程序坞图标，程序最小化：

```bash
gsettings set org.gnome.shell.extensions.dash-to-dock click-action 'minimize'
```

音视频编解码器：

```bash
sudo apt-get install ubuntu-restricted-extras
```

音频控制：

```bash
sudo apt install pulseaudio pavucontrol
```

gnome 桌面插件：

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

## 参考

1. [How to install and manage fonts on Linux](https://linuxconfig.org/how-to-install-and-manage-fonts-on-linux)
2. [GCC-13.2.0 编译安装](https://cuterwrite.top/p/gcc-13-source-install/)
3. [在 Ubuntu 安装配置 Fcitx 5 中文输入法](https://muzing.top/posts/3fc249cf/)
4. [The update-alternatives Command in Linux](https://www.baeldung.com/linux/update-alternatives-command)
5. [VirtualBox: linux 没有权限访问共享文件夹的问题](https://www.cnblogs.com/yongdaimi/p/13424855.html)
