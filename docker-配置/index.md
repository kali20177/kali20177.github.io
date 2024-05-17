# Docker 基础


## 安装

Windows 上 Docker 需要基于 Hyper-V 或者 WSL2。在“启用或关闭 Windows 功能”中勾选 "Hyper-V" 和“容器”，进而再安装 Docker Desktop。

安装 Docker Desktop 后可以 Settings --> Resources --> WSL Integration 选择 Enable integration with additional distros。这样 WSL2 中即可直接使用 Windows 上安装的 Docker 环境。

![wsl_docker](/Docker_img/wsl_docker.png)

Linux 上可以直接使用 apt 的源下载 Ubuntu 维护的社区版。

```bash
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release usbutils
sudo apt install docker.io docker-compose docker-doc
```

安装完成后，docker 默认 root 用户才能访问，所以要添加当前用户名到 docker 组中。

```bash
sudo usermod -aG docker ${USER}
```

执行后需要重启，重启后输入以下命令查看用户名是否被添加到组中：

```bash
id -nG
```

现在直接执行 docker 命令就不会报错。

## 概念

- image:
- container:
- dockerfile:

## 基础命令

首先建议换阿里源，如果在 Linux 命令行下使用，可以通过修改 daemon 配置文件 `/etc/docker/daemon.json` 实现，保存重新加载 daemon 并重启容器。

```
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xxx.mirror.aliyuncs.com"],
  "insecure-registries": [
    "ip:port"
  ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

使用 Docker Desktop 则在 settings 的 `Docker Engine` 中直接编辑。

保存和加载完整镜像：

```bash
docker save busybox > busybox.tar # linux
docker save --output busybox.tar busybox

docker load < busybox.tar.gz # linux
docker load --input fedora.tar
```

## UI 工具

- docker desktop
- lazydocker

## SSH 以及基础编译环境配置

### Ubuntu

ubuntu 默认镜像拉取后需要先使用默认源安装 ca-certificates，之后才能换源。

由于容器默认没有文本编辑器，可以使用 cat 或者 echo 用重定向标准输入的方式直接覆盖源文件。

```bash
cat >/etc/apt/sources.list
# 获取输入内容...，Ctrl + d 结束
```

安装 OpenSSH 服务，设置 root 密码

```bash
apt install openssh-server
passwd:
# 修改密码
```

OpenSSH 的客户端软件是 ssh，服务器软件是 sshd，服务器默认监听端口是 22。启动镜像时分配端口号映射，把本机的 6789 端口映射到容器 22 号端口：

```bash
docker run -itd -p 6789:22 image-name
```

Windows powershell 下可以使用 `netstat -a` 查看端口 6789 是否被正常打开。使用 exec 和运行中的容器交互：

```bash
docker exec -it container-id /bin/bash
```

修改容器中的 sshd_config 配置文件，开启密码访问：

```bash
vim /etc/ssh/sshd_config
#PermitRootLogin prohibit-password   在此行后添加
PermitRootLogin yes
```

在容器中执行：

```bash
service ssh start
# 或者
/etc/init.d/ssh start
```

此时本机终端可以连接容器内部。

```powershell
ssh root@127.0.0.1 -p 6789
```

> 对于 Windows，系统变量下默认的 ssh 工具连接会存在问题，建议更换成 git 安装目录下 /usr/ssh.exe

对于 VSCode，安装插件 Remote SSH 后，选择**将当前窗口连接到主机**，填写 `ssh root@127.0.0.1 -p 6789`，输入密码即可连接完成。

使用 ssh 复制文件：

```bash
# 远程复制到本机，scp 命令已经不再推荐
scp -p 22 username@ipaddress:/home/xxx/ .
# 复制文件到远程
rsync -p 22 filename username@ipaddress:/home/xxx/ 
```

### alpine

alpine 发行版更加轻量，并且不需要安装 ca-certificates 换源，比较容易写 Dockerfile。alpine 使用 apk 作为包管理工具，且没有 bash。基础编译工具通过 `apk add build-base` 安装。

```sh
apk add openrc openssh iptables
```

然后执行：

```sh
rc-update add sshd
rc-status  # 确认 sshd 监听状态
rc-service sshd start
```

其他步骤和 ubuntu 中相同。

### Fedora

RedHat 维护的发行版，默认安装的编译环境较新。使用 dnf 作为包管理器。但是默认甚至不支持 `clear` 命令。

```bash
sudo dnf group install "C Development Tools and Libraries" "Development Tools"
sudo dnf install ncurses   # 支持 clear
sudo dnf install clang-tools-extra   # 安装 clangd
```

## 私有 Registry

为了在 Dockerfile 中使用自定义的镜像，并且分享镜像到局域网，需要使用 Docker Registry 容器。

```bash
docker pull registry
docker run --name MyHub -d -p 5000:5000 -v /f/docker_registry:/var/lib/registry registry
```

启动时添加映射端口，添加局域网或本地存放仓库的位置到容器路径 `/var/lib/registry` 的映射。启动后需要给要上传的镜像打上标签，这里以镜像 alpine:cpp 为例：

```bash
docker tag alpine:cpp your_ip:5000/alpine:cpp
```

上传之前先添加局域网 ip:port 到 Docker Engine 的 json 文件中，否则会上传失败。

```json
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  # 添加此段
  "insecure-registries": [
    "ip:port",
    "ip:port"
  ]
}
```

点击应用并重启镜像，上传：

```bash
docker push your_ip:5000/alpine:cpp
```

只要局域网内其他人也添加了相同的 `"insecure-registries"` 字段，就能访问 registry 容器内的自定义镜像。下载命令：

```bash
docker pull your_ip:5000/alpine:cpp
```

如果局域网内有多个镜像，可以使用 curl 查看本地镜像列表：

```bash
curl your_ip:5000/v2/_catalog
```

如果只是本机使用，可以在 tag 和 push 阶段直接使用主机环回地址：

```bash
docker tag ubuntu:latest 127.0.0.1:5000/ubuntu:latest
```

这种情况下无需编辑 Docker Engine 中的 json 文件即可拉取自定义镜像。

## GUI 

### Windows

在 Windows 下安装带有 X11 服务的软件，例如 X410，VcXsrv，MobaXterm 等。

![x-display](/Docker_img/x-display-server-overview.png)

编写下列的 Dockerfile，以简单钟表程序 xarclock 为例：

```dockerfile
FROM ubuntu:18.04
# 换源操作略
# 安装 xarclock
RUN apt-get update && apt-get install -y xarclock
CMD /usr/bin/xarclock
```

使用以下命令构建：

```bash
docker build -t xarclock .
docker run -ti --rm -e DISPLAY=host.docker.internal:0.0 xarclock
```

Docker 推荐容器使用 host.docker.internal 来访问宿主机上的服务。输出乱码需要在容器中安装对应语言、地区和字体。

### Linux

Linux 同样通过 x11 服务显示，启动参数有所不同：

```bash
sudo apt install x11-xserver-utils
sudo xhost +
# access control disabled, clients can connect from any host
docker run -it 
    -v /etc/localtime:/etc/localtime:ro \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -e DISPLAY=unix$DISPLAY \
    -e GDK_SCALE \
    -e GDK_DPI_SCALE \
    --name GUITEST ubuntu zsh
```

## 嵌入式 Debug

Windows 下 WSL 环境的容器，目前对宿主机设备的访问支持不完善，无法直接调试。Linux 主机上的容器可以通过设备映射的方式 Debug。

### 方法一

J-Link 官方提供了在 Linux 宿主机上直接用设备映射访问调试器的文档，容器内需要安装 `lsusb` 支持。

```bash
apt install usbutils
sudo docker run -t -i --privileged -v /dev/bus/usb/:/dev/bus/usb ubuntu bash
```

### 方法二

在 Windows 的容器环境下，比较容易做到的方法是使用 GdbServer 进行远程调试。

![docker-jlink](/Docker_img/docker-jlink.png)

> gdbserver is a control program for Unix-like systems, which allows you to connect your program with a remote GDB via target remote or target extended-remote—but without linking in the usual debugging stub.

在使用 cortex-debug 插件和 J-Link 的情况下，在 launch.json 中添加如下配置：

```json
{
    "name": "Remote GdbServer",
    "request": "launch",
    "type": "cortex-debug",
    "cwd": "${workspaceFolder}",
    "executable": "./build/xxx.elf",
    "runToEntryPoint": "main",
    "showDevDebugOutput": "none",
    "servertype": "external",
    "preRestartCommands": [
        "file ./build/xxx.elf",
        "load",
        "enable breakpoint",
        "monitor reset",
        "monitor halt"
      ],
    "gdbPath": "gdb-multiarch",
    "objdumpPath": "arm-none-eabi-objdump",
    "gdbTarget": "host.docker.internal:2331" 
}
```

调试前在 Windows 端打开 J-LinkGDBServer.exe，选择设备和接口类型。arm-none-eabi-gdb 在容器中用 tcp 通过端口 2331 实现和宿主机中 J-LinkGDBServer 的通信。从而实现正常的调试功能。

Linux 下如果要使用图形化的 JlinkGdbServer, 需要下载 libqtgui4

使用 openocd 也能实现类似的功能。

### 方法三

也可以直接使用 J-Link Remote Server，配置正确的 IP 地址实现调试。以 LAN mode 在宿主机启动 J-Link Remote Server。添加如下的调试配置：

```json
{
    "name": "Remote IP",
    "request": "launch",
    "type": "cortex-debug",
    "cwd": "${workspaceFolder}",
    "executable": "./build/xxx.elf",
    "device": "xxx",
    "interface": "swd",
    "runToEntryPoint": "main",
    "showDevDebugOutput": "none",
    "servertype": "jlink",
    "preRestartCommands": [
        "file ./build/xxx.elf",
        "load",
        "enable breakpoint",
        "monitor reset",
        "monitor halt"
      ],
    "gdbPath": "gdb-multiarch",
    "objdumpPath": "arm-none-eabi-objdump",
    "serverpath": "JLinkGDBServerCLExe",
    "ipAddress": "xxx.xxx.xxx.xxx",
}
```

即直接使用本地的 JLinkGDBServerCLExe 服务，通过实际的 IP 地址远程调试。实际使用中经常断连，不推荐这样调试。

## Harbor

Harbor 是一个开源的镜像仓库管理方案，集成了反向代理，数据库等功能。提供了网页的管理界面，能实现对镜像的访问控制，漏洞扫描等功能。可以在 WSL2 或者各种 Linux 发行版部署，目前不适用于 ARM 系列芯片的 MacOS 系统。

![Harbor_Architecture](/Docker_img/Harbor_Architecture.png)

下载并解压 offline 版本的安装包后，首先完成配置文件。

```bash
cp harbor.yml.tmpl harbor.yml
vim harbor.yml
```

修改 hostname 为对外访问的 IP 地址。默认 80 端口，暂时注释掉 HTTPS 部分的配置。

```yml
# The IP address or hostname to access admin UI and registry service.
# DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
hostname: ip_addr

# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 80

# https related config
# https:
  # https port for harbor, default is 443
  # port: 443
  # The path of cert and key files for nginx
  # certificate: /your/certificate/path
  # private_key: /your/private/key/path
  # enable strong ssl ciphers (default: false)
  # strong_ssl_ciphers: false
```

配置完成后即可执行 `sudo ./install.sh` , **执行安装前**建议先像配置 Registry 一样将 `"insecure-registries": ["ip_addr:80"] ` 添加到 Docker Engine 的 json 文件中。

docker compose 服务启动后，可以在浏览器输入 `ip_addr:80` 访问 harbor 网页端，默认用户名 admin，密码：Harbor12345。

![Harbor_login](/Docker_img/Harbor_login.png)

通过管理界面创建一个 push 镜像需要登陆，pull 没有限制的仓库，通过 **新建项目** 添加。

![public_hub](/Docker_img/public_hub.png)

下面将制作好的镜像上传到新建的 public 仓库，首先按照下面的**格式**修改镜像 tag：

```bash
docker tag fedora:cpp ip_addr:80/public/fedora:cpp
```

push 之前先在终端登录 Harbor：

```bash
> docker login ip_addr:80
Username: admin
Password: 
Login Succeeded

> docker push ip_addr:80/public/fedora:cpp
```

这样在网页后台就能看到上传的镜像，其他人可直接复制 pull 镜像的指令。

## VSCode

容器和 VSCode 可以良好结合，需要安装插件 Docker 和 Dev Containers。进入 VSCode 点击远程可以选择附加到运行的容器打开。

> 注意当首次进入容器时，VSCode 会在容器中安装 code-server 服务。如果当前网络环境不允许容器联网，需要手动拷贝 vscode-server 到容器内。在 "关于"中查看当前 VSCode 的“提交”commit-id，2023 年之前可以访问 `https://update.code.visualstudio.com/commit:${commit_id}/server-linux-x64/stable` , 2023 年之后访问 `https://vscode.download.prss.microsoft.com/dbazure/download/stable/${commit_id}/vscode-server-linux-x64.tar.gz`。

code-server 服务安装的位置和容器的用户名有关，例如对于微软的 [mcr](https://github.com/devcontainers/images/tree/main/src) 系列容器镜像，在 `/home` 下定义了一个名为 vscode 的非 root 用户。因此 code-server 会被安装在 `/home/vscode/.vscode-server/bin/commit-id`。而普通容器默认用户是 root，那么 code-server 会安装在 `/root/.vscode-server/bin/commit-id`。

## 参考

1. [Docker - Docker for Windows 10 入門篇](https://skychang.github.io/2017/01/06/Docker-Docker_for_Windows_10_First/)
2. [镜像加速器](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)
3. [SSH 服务器](https://wangdoc.com/ssh/server)
4. [Alpine Linux 常用命令](https://www.cnblogs.com/jackadam/p/9290366.html)
5. [Setting up a SSH server](https://wiki.alpinelinux.org/wiki/Setting_up_a_SSH_server?ref=angelsanchez.me)
6. [通过 SSH 在远程和本地系统之间传输文件的 4 种方法](https://zhuanlan.zhihu.com/p/507876254)
7. [docker 搭建本地/局域网仓库](https://juejin.cn/post/7248827630866415674)
8. [私有仓库](https://docker-practice.github.io/zh-cn/repository/registry.html)
9. [about – x11 display server – display or login manager – window manager](https://dwaves.de/2017/06/14/about-x11-display-server-display-or-login-manager-window-manager/)
10. [在 Docker for Windows 中运行 GUI 程序](https://www.cnblogs.com/larva-zhh/p/10531824.html)
11. [Run GUI app in linux docker container on windows host](https://dev.to/darksmile92/run-gui-app-in-linux-docker-container-on-windows-host-4kde)
12. ["error: XDG_RUNTIME_DIR not set in the environment." when attempting to run nautilus as root](https://askubuntu.com/questions/456689/error-xdg-runtime-dir-not-set-in-the-environment-when-attempting-to-run-naut)
13. [Docker 容器图形界面显示（运行 GUI 软件）的配置方法](https://www.cnblogs.com/ruiyang-/p/10185840.html)
14. [J-Link Docker Container](https://wiki.segger.com/J-Link_Docker_Container)
15. [Win10 中的 Docker 使用 USB 设备](https://www.voidking.com/dev-win10-docker-usb/)
16. [Using the gdbserver Program](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Server.html)
17. [Developing on the RP2040 / Pi Pico with Docker and JLink](https://johnmcnelly.com/developing-on-the-rp2040-pi-pico-with-docker-and-jlink/)
18. [vscode + jlink + GDBServer 在线调试](https://blog.csdn.net/niu_88/article/details/127347197)
19. [J-Link GDB Server](https://wiki.segger.com/J-Link_GDB_Server)
20. [STM32 Development Env for Windows: VSCode + ARM GCC Toolchain + OpenOCD](https://mightydevices.com/index.php/2019/09/stm32-development-env-for-windows-vscode-arm-gcc-toolchain-openocd/)
21. [WSL](https://wiki.segger.com/WSL)
22. [Run the Installer Script](https://goharbor.io/docs/2.10.0/install-config/run-installer-script/)
23. [VS Code Server 离线安装过程](https://zhuanlan.zhihu.com/p/294933020)
