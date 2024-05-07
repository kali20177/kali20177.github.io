# Gitee 使用问题

## 1. SSH 密钥

### 1.1 问题

使用 git push 时出现：

```bash
git@gitee.com: Permission denied (publickey).
fatal: 无法读取远程仓库。

请确认您有正确的访问权限并且仓库存在。
```

### 1.2 解决

对自己的邮箱，重新生成 SSH key

```bash
ssh-keygen -t rsa -C "XXX@XX.com" 
```

一直回车，得到类似：

```bash
The key fingerprint is:
SHA256:1jZz2AqNZQJNpWRf0miTxYU4EAmkT+Ip1P5lbZRqdN0 XXX@XX.com
The key's randomart image is:
+---[RSA 3072]----+
|     .++*=oBoo.  |
|   . . +ooB=+.   |
|  . + . +.Bo. E  |
| . o = . % o     |
|  . + . S X     |
|   . . = + =     |
|      .   .      |
|                 |
|                 |
+----[SHA256]-----+
```

查看生成的密钥

```bash
vim ~/.ssh/id_rsa.pub
```

复制密钥，添加到 [gitee](https://gitee.com/profile/sshkeys)

终端输入：

```bash
ssh -T git@gitee.com 
```

出现：

```bash
Hi kali_mufasa! You've successfully authenticated, ...
```

完成。后半句警告 "but GitHub does not provide shell access" 没有关系。

## 2. 避免用户名和密码的输入

首先查看当前连接情况

```bash
git remote -v
```

如果输出是

```bash
origin  https://gitee.com/xxx/xxx.git (fetch)
origin  https://gitee.com/xxx/xxx.git (push)
```

说明使用的是 https 连接方式，需要反复输入用户名和密码，修改为 SSH 连接更安全。

```bash
git remote rm origin
git remote add origin git@gitee.com:xxx/xxx.git
```

