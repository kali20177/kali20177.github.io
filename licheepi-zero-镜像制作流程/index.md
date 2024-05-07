# Licheepi Zero 镜像制作流程


licheepi-zero 采用全志 V3s 芯片，是一款小型的 Linux 开发板。官方提供了镜像可以直接使用 Win32DiskImager 烧录，也可以根据需要自己制作。和大多嵌入式 Linux 系统一样主要分为 uboot 系统引导，linux 内核以及根文件系统三个部分。本文主要记录制作过程中可能遇到的问题和教程中缺失的部分。

## 1. 前期准备

**知识：**

1. Linux 常用命令
2. Makefile 的基本概念

**设备：**

* usb-ttl 模块
* licheepi-zero 开发板
* sd 卡
* 4.3 寸通用屏
* linux 系统 (实体机或者虚拟机，用于编译和烧录 SD 卡)

**工具链：**

1. 交叉编译器 `arm-linux-gnueabihf`

    ```bash
    wget https://releases.linaro.org/components/toolchain/binaries/latest/arm-linux-gnueabihf/gcc-linaro-6.3.1-2017.05-x86_64_arm-linux-gnueabihf.tar.xz
    ```

    解压后放到某处，并在 zshrc 或者 bashrc 中添加环境变量。

    ```bash
    export PATH=$PATH:/usr/local/arm/gcc-linaro-6.3.1-2017.05-x86_64_arm-linux-gnueabihf/bin
    ```

2. 编译过程中依赖的包

    ```bash
    sudo apt install git wget make gcc g++ flex bison libssl-dev bc kmod automake autoconf libtool
    sudo apt installl linux-headers-$(uname -r)
    sudo apt installl libncurses5-dev
    sudo apt-get install device-tree-compiler
    sudo apt-get install patch cpio unzip rsync 
    ```

3. Python 2 环境

    实际使用过程中发现，在编译根文件系统时会用到系统的 Python，并且是 Python2 脚本，所以需要确保系统的环境变量 `python` 指向的是 `python2.7`。

    ![python2](/Licheepi-image/python2.png)

## 2. 编译系统

### 2.1 uboot

uboot 中相关的配置有 Soc 架构、DDR、LCD、时钟和 SPL 等，详细内容和参看官方文档。

下载 uboot:

```bash
git clone https://github.com/Lichee-Pi/u-boot.git -b v3s-current
```

**此时需要修改配置文件**，使 u-boot 可以直接从 tf 卡启动，在 u-boot 目录的 `include/configs/sun8i.h` 文件中添加

```c
#define CONFIG_BOOTCOMMAND   "setenv bootm_boot_mode sec; " \
                            "load mmc 0:1 0x41000000 zImage; "  \
                            "load mmc 0:1 0x41800000 sun8i-v3s-licheepi-zero-dock.dtb; " \
                            "bootz 0x41000000 - 0x41800000;"

#define CONFIG_BOOTARGS      "console=ttyS0,115200 panic=5 rootwait root=/dev/mmcblk0p2 earlyprintk rw  vt.global_cursor_default=0"
```

编译 (这里使用 4.3 寸屏幕):

```bash
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- LicheePi_Zero_480x272LCD_defconfig 
make ARCH=arm menuconfig
time make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- 2>&1 | tee build.log
```

编译完成后结果在当前目录，文件名 `u-boot-sunxi-with-spl.bin`。

### 2.2 kernel

下一步编译 Linux 内核，首先下载：

```bash
git clone -b zero-5.2.y --depth 1 https://github.com/Lichee-Pi/linux.git
```

进入 Linux 文件夹，修改 Makefile 文件，找到 364 行写入：

```text
[364]ARCH            = arm
[365]CROSS_COMPILE   = arm-linux-gnueabihf-
[366]INSTALL_MOD_PATH = out
```

开始编译：

```bash
make licheepi_zero_defconfig
make menuconfig
make -j4
make dtbs
```

完成后 zImage 文件在`arch/arm/boot/`下，设备树在`arch/arm/
boot/dts`下。

**首次编译一般都会遇到如下问题：**

```bash
/usr/bin/ld: scripts/dtc/dtc-parser.tab.o:(.bss+0x20): multiple definition of `yylloc'; scripts/dtc/dtc-lexer.lex.o:(.bss+0x0): first defined here
collect2: error: ld returned 1 exit status
make[1]: *** [scripts/Makefile.host:99: scripts/dtc/dtc] Error 1
```

需要对源文件修改几处：

```bash
vim scripts/dtc/dtc-parser.tab.h
修改extern XXX yylloc 为yyloc
vim scripts/dtc/dtc-lexer.l
修改XXX yylloc 为 extern XXX yylloc
```

### 2.3 rootfs

下载 buildroot：

```bash
wget https://buildroot.org/downloads/buildroot-2017.08.tar.gz
```

配置根文件系统：

```bash
make menuconfig
```

比较重要的包括：

* 配置文件保存位置
* 外部工具链
* C library
* hostname
* Root password
* Target package(常用软件库)

**在选择常用软件库时，部分库需要在 configmenu 中选中 C++ 支持才能选中**，例如 Qt。

![Cpp_support](/Licheepi-image/Cpp_support.png)

直接使用 make 命令编译。编译过程中会联网下载选中的包，部分托管在 github 等境外服务器的包可能会遇到网络问题。此时可暂停编译，复制当前下载包的地址，通过 ghproxy 代理手动下载到 dl 目录下，再执行编译将自动检测到目标。
编译完成后会在 `output/images` 下生成 rootfs.tar。

## 3. 镜像制作

### 3.1 SD 卡分区和挂载

在 Linux 安装 Gparted 图形化分区工具。

```bash
sudo apt install gparted
```

通过 `ls /dev/sd*` 命令查看当前插入 sd 卡名称。

将 SD 卡原本的分区删掉，建立一个 32M FAT16 格式 (或 FAT32) 的分区存放引导文件和系统镜像，用剩余部分建立 ext4 格式分区。

![fenqu](/Licheepi-image/fenqu.png#right)

重新插拔 SD 卡，需要确认两个分区是否挂载成功：

```bash
df -h
```

实际使用过程中，一般两个分区无法同时自动挂载成功。
如果存在未被挂载的情况需要手动挂载，首先创建挂载文件夹 BOOT 和 rootfs(一般在 `/media` 或者 `/mnt` 下)，然后使用`mount`命令挂载 (umount 取消挂载）:

```bash
sudo mount /dev/sda1 /media/mufasa/BOOT
sudo mount /dev/sda2 /media/mufasa/rootfs
```

### 3.2 复制文件

将四个文件 rootfs.tar，sun8i-v3s-licheepi-zero-dock.dtb，u-boot-sunxi-with-spl.bin，zImage 复制到同一文件夹中。

1. 使用 dd 命令将 uboot 文件写到 8k 位置 (规定)

    ```bash
    sudo dd if=u-boot-sunxi-with-spl.bin of=/dev/sdx bs=1024 seek=8
    ```

2. 将 dtb 和 zImage 拷贝到 fat16 分区。

    ```bash
    cp sun8i-v3s-licheepi-zero-dock.dtb /media/mufasa-/BOOT/
    cp zImage /media/mufasa/BOOT/
    ```

3. 根文件系统解压到 ext4 分区

    ```bash
    sudo tar xvf rootfs.tar -C /media/mufasa/rootfs/
    ```

    默认使能串口登录，需要修改 /etc/inittab :

    ```text
    #console::respawn:/sbin/getty -L  console 0 vt100 # GENERIC_SERIAL
    ttyS0::respawn:/sbin/getty -L ttyS0 115200 vt100 # GE
    ```

    需要免密码登录，直接

    ```text
    #console::respawn:/sbin/getty -L  console 0 vt100 # GENERIC_SERIAL
    ttyS0::respawn:/bin/sh
    ```

## 4. 效果

打开 MobaXterm 新建一个串口会话，波特率 115200。使用杜邦线按照如下方式连接开发板和 usb-ttl 模块：

```text
usb-ttl --> 开发板引脚
3.3V --> 供电
GND --> GND 
RX  --> TX(U0) 
TX  --> RX(U0)
```

打开串口后能看到屏幕被点亮，并通过电脑串口能够登录和操作开发板。

![licheepi_zero_disp](/Licheepi-image/licheepi_zero_image_disp.png)

## 5. 参考链接

1. [licheepi 官方文档](https://wiki.sipeed.com/soft/Lichee/zh/Zero-Doc/Start/intro_cn.html)
2. [荔枝派 Zero V3s 开发板入坑记录](https://whycan.com/t_561.html)
3. [荔枝派 zero 镜像制作及烧录](https://blingblingxuanxuan.github.io/2021/03/07/lichee-start/)

