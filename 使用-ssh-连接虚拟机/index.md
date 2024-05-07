# SSH 连接虚拟机


## 1. 在本地和虚拟机中都安装 OpenSSH

```bash
sudo apt install openssh-server
```

> 测试是否安装 ssh
>
> ssh localhost

## 2. 编辑本机 SSH config 文件打开默认端口

```bash
vim /etc/ssh/sshd_config
```

![sshd 端口](/SSH_VBOX/sshd_port.png)

## 3. 虚拟机设置

>### 网络地址转换 NAT

`设置` --> `网络` --> `网络地址转换` --> `高级` --> `端口转发` --> `添加规则` ：

![端口转发.png](/SSH_VBOX/Port_forwarding.png)

1. 名称可随便填写，如 ssh
2. 主机 IP 可以不填，或者填写 127.0.0.1
3. 主机端口随意填写一个不会产生冲突的端口，如 2222
4. 子系统端口可以不填，或者填写子系统当前 IP 地址
5. 子系统端口如果是要进行 ssh 连接，则填写 22；ftp 就填写 21

## 4. 本地和虚拟机重启 ssh

```bash
sudo /etc/init.d/ssh restart
```

> 其他：
>
> `sudo /etc/init.d/ssh start   #启动ssh服务`
> `sudo /etc/init.d/ssh stop    #停止ssh服务`

## 6. 连接

```bash
ssh -l [username] -p 1111 127.0.0.1
```

根据提示输入密码和用户名。结果：

![ssh 结果](/SSH_VBOX/ssh_res.png)

输入`exit`可退出。

## 参考

1. [使用 ssh 连接 VirtualBox 虚拟机](https://www.jianshu.com/p/d59ed9f226d1)

2. [Linux——本机终端 SSH 连接 VirtualBox 中的 Linux 虚拟机](https://www.cnblogs.com/oddcat/articles/9676817.html)

