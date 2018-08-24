---
title: 【SSH】使用SSH登录远程主机，并禁用密码登录
date: 2018-07-30 16:54:08
tags:
  - ssh
category:
  - Linux
---

对远程主机进行登录管理，一方面可以简化日常频繁登录的密码和 ip 输入步骤，另一方面，也可以提高远程主机的安全性，避免远程主机被“黑客”轻易攻击。
也借此加强对 Linux 文件权限的认识和理解。

<!--more-->

## 本地生成 SSH 秘钥

### 生成本机系统的 ssh 公钥

```bash
ssh-keygen
```

默认保存路径为 `~/.ssh/`

[windows 对应为`C:\Users\Administrator\.ssh\`][mobaxterm对应为`/home/mobaxterm/.ssh/`]

```bash
# 公钥路径
~/.ssh/id_rsa.pub

# 私钥路径
~/.ssh/id_rsa
```

### 复制公钥

```bash
cat ~/.ssh/id_rsa.pub
```

## 远程主机配置 ssh

### 使用密码登录到远程主机

```bash
ssh {登录用户}@{ip地址}

输入密码后进入远程主机系统
```

### 查看远程主机的 ssh 配置

配置文件目录`/etc/ssh/`

```bash
/etc/ssh/sshd_config
```

对以下参数进行设置：

```ini
# 默认的认证公钥文件
AuthorizedKeysFile .ssh/authorized_keys
```

将`本地的公钥`复制到`远程的公钥认证文件` [~/.ssh/authorized_keys]中保存

### 设置文件和目录权限

【理论说明】：

**一栏有十个字符**：

- 第一个字符用于标识是`文件`还是`目录`

- 后面九个字符为三组：
  - 第一组为『文件拥有者的权限』；
  - 第二组为『同群组的权限』；
  - 第三组为『其他非本群组的权限』。

其中：

[ r ]代表可读(read)，值为 4 [二进制：100][ w ]代表可写(write)，值为 2 [二进制：010][ x ]代表可执行(execute)，值为 1 [二进制: 001][ - ]代表没有操作权限，值为 0 [二进制： 000]

参考来源: [http://cn.linux.vbird.org/linux_basic/0210filepermission.php](http://cn.linux.vbird.org/linux_basic/0210filepermission.php)

（1）设置`.ssh`目录权限

```bash
# 文件拥有者拥有读、写、执行权限,其他组无权限
chmod 700 ~/.ssh/
```

（2）设置`authorized_keys`文件权限

```bash
# 文件拥有者拥有读、写权限，同群组和其他群组成员拥有读权限
chmod 644 ~/.ssh/authorized_keys
```

### 对 ssh 进行配置

```bash
# 位置：/etc/ssh/sshd_config

# 允许root用户通过ssh登录
PermitRootLogin yes

# 允许使用ssh权限登录
RSAAuthentication yes
PubkeyAuthentication yes
```

使用秘钥方式登陆后,禁用密码登录[之前密码登录的 session 将失效]

```bash
# 禁用密码登录
PasswordAuthentication no
```

重启 ssh

```bash
service sshd restart
```

参考来源: [https://hyjk2000.github.io/2012/03/16/how-to-set-up-ssh-keys/](https://hyjk2000.github.io/2012/03/16/how-to-set-up-ssh-keys/)

## 本地 ssh 配置进行快捷登录

### 配置 ssh

```bash
# ~/.ssh/config
# 定义登录远程主机的ssh连接名
Host txyun

# 定义远程主机ip地址
HostName {ip地址}

# 定义远程主机的ssh端口号[默认情况下ssh端口号为22]
Port 22

# 设置登录用户名, root用户拥有所有权限
User root
```

### 进行 ssh 方式登录远程主机

```bash
ssh txyun
```

即可直接登录。

## 总结

简化了可信访客的登录步骤，并且也只有和远程服务器 ssh 认证文件中公钥相匹配的主机才能成功登录，提高了服务器的安全性。

完。
