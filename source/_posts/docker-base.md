---
title: 【Docker】入门使用
date: 2018-07-31 13:00:00
tags:
  - docker
category:
  - 【Docker相关】
---

在学习 Docker 的镜像源设置和安装配置后，开始学习 Docker 的基本使用，包括服务进程的管理，容器、镜像的基本使用和管理。

<!--more-->

## Docker 使用命令

### 系统管理相关

```bash
# 查看docker运行状态
sudo service docker status
sudo systemctl docker status

# 启动和停止docker
sudo service docker start
sudo systemctl docekr start

sudo service sdocker stop
sudo systemctl docekr stop
```

### Docker 运行

(1)下载 Docker Hub 上的"hello-world"镜像[大小只有 1.85k]

```bash
sudo docker run hello-world
```

(2) 查看 docker 镜像

```bash
docker images
```

(3) 查看 docker 容器

```bash
# 查看运行状态的container
docker ps

# 查看所有的container
docker ps -a
```

(4)下载`ubuntu`镜像，创建容器，保持可写入操作，并以 bash 状态进入容器

```bash
sudo docker run -i -t ubuntu /bin/bash
# [-i]: 保证容器中STDIN是开启的，持久的标准写入
# [-t]: 为要创建的容器分配一个伪tty终端，使新创建的容器能够提供一个交互式shell
# [ubuntu]: 使用的镜像名，如果不存在则下载Docker Hub上的镜像
# [/bin/bash]: 让新建的容器运行/bin/bash命令
```

只有指定的/bin/bash 命令处于运行状态的时候，容器才处于运行状态，一旦 exit 退出容器，/bin/bash 命令也就结束了，这时容器也随之停止运行。
但容器仍然是存在的。

Docker 在文件系统内部用这个镜像创建了一个新容器，该容器拥有自己的网络、IP 地址，以及一个用来和宿主机进行通信的桥接网络接口。

### 容器的基本使用

(1)查看容器的主机名[即该容器的 ID]

`hostname`

未完待续！
