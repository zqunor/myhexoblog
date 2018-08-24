---
title: 【Docker】--安装与配置
date: 2018-07-31 11:00:00
tags:
  - docker
category:
  - Docker
---

docker 是 linux 虚拟化技术，能够一键式搭建开发环境，并且能保证运维、开发、上线部署的环境完全一致，避免了运行环境差异性带来的问题。
具有简单、轻量、快速、高效的特性。掌握 Docker 的安装和相关配置也是提升开发技能的重要途径。

<!--more-->

## Linux 安装[ubuntu/centos]

### 安装前检查

**Ubuntu > 12**

(1).内核版本：[>=Linux 3.8]

```bash
uname -a
```

(2).存储驱动 Device Mapper[文件存在即已安装]

```bash
ls -l /sys/class/misc/device-mapper

# 如果不存在，加载dm_mod模块
sudo modprobe dm_mod
```

(3).开启[cgroup](http://en.wikipedia.org/wiki/Cgroups)和命名空间(namespace)功能

2.6.38 以后的内核都已经提供了良好的支持。

(4).系统为 64 位[x86_64 和 amd64] 【目前不支持 32 位 CPU】

### 安装方式

（1）安装 Ubuntu/CentOS 维护的版本

[ubuntu]

```bash
# 安装docker[ubuntu]
sudo apt-get install -y docker.io

# 配置docker
source /etc/bash_completion.d/docker.io
```

[centos]

```bash
# 安装docker[centos]
sudo yum install -y docker
```

（2）安装 Docker 维护的版本

```bash
# 1.检查APT的HTTPS支持，查看/usr/lib/apt/methods/https文件是否存在，如果不存在，运行安装命令
apt-get update
apt-get install -y apt-transport-https

# 2.添加Docker的apt仓库
echo deb https://get.docker.com/ubuntu docker main > /etc/apt/sources.list.d/docker.list

# 3.添加仓库的key
apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys {你的key}

# 4.安装
apt-get update
apt-get install -y lxc-docker
```

（3）脚本安装：

```bash
# 查看系统是否安装curl
whereis curl

# 安装curl命令
sudo apt-get install -y curl   [ubuntu]
sudo yum install -y curl    [centos]

# 运行脚本 [获得当前最新稳定版docker]
curl -sSL https://get.docker.com/ubuntu/ | sudo sh
```

（4）查看 docker 版本信息

```bash
# 查看docker版本
sudo docker version
```

当前版本信息：

```bash
Client:
 Version:         1.13.1
 API version:     1.26
 Package version: docker-1.13.1-68.gitdded712.el7.centos.x86_64
 Go version:      go1.9.4
 Git commit:      dded712/1.13.1
 Built:           Tue Jul 17 18:34:48 2018
 OS/Arch:         linux/amd64

Server:
 Version:         1.13.1
 API version:     1.26 (minimum version 1.12)
 Package version: docker-1.13.1-68.gitdded712.el7.centos.x86_64
 Go version:      go1.9.4
 Git commit:      dded712/1.13.1
 Built:           Tue Jul 17 18:34:48 2018
 OS/Arch:         linux/amd64
 Experimental:    false
```

### 查看 docker 运行效果

```bash
sudo docker run ubuntu echo 'Hello World'
```

```info
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```

### 使用非 root 用户[为当前用户添加 docker 用户组]

默认使用 docker 相关命令时，必须使用 root 权限，实际上可以创建 docker 用户组，从而简化 docker 使用操作，不用在所有的 docker 命令前使用`sudo`命名。

```bash
# 1.添加docker用户组
sudo groupadd docker

# 2.将当前用户添加到docker用户组
sudo gpasswd -a ${USER} docker
# 或
sudo usermod -aG docker $USER

# 3.重启docker服务
sudo service docker restart
```

[注意，执行 gpasswd 命令之后需要注销后重新再登录才能生效]

> 添加到 docker 用户组存在安全隐患，docker 用户组对 Docker 具有与 root 用户相同的权限，
> 所以 docker 用户组中应该只能添加确实需要使用 Docker 的用户和程序

### Docker 升级

[ubuntu]

```bash
sudo apt-get update
sudo apt-get install docker
```

[centos]

```bash
sudo yum update
sudo yum install docker
```

## 安装 docker-compose

### pip 安装

```bash
sudo pip install docker-compose
```

### 下载二进制文件安装

【查看最新版版本号】:[docker/compose--github](https://github.com/docker/compose/releases)

```bash
# 下载最新版的docker-compose[版本号需要前往github确认是否为最新，1.22.0是目前的最新版2018/7/31]
sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

# 对docker-compose添加可执行权限
sudo chmod +x /usr/local/bin/docker-compose

# 查看版本信息
sudo docker-compose -v
```

## Windows 安装

### Docker 基本介绍

- Linux 容器技术
- 操作系统级别的虚拟化
- 依赖于 Linux 内核的 Namespace 和 Cgroups

### windows 下的 docker[Boot2Docker for windows|Docker ToolBox]

- Boot2Docker Linux ISO | Docker ToolBox [为 docker 定制的虚拟机镜像，包含 docker 的运行环境]
- Virtualbox [提供虚拟机服务的软件]
- MSYS-git [提供 shell 运行环境]
- 管理工具
