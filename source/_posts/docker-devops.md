---
title: 【Docker】系统学习 Docker 践行 DevOps 理念
date: 2018-12-11 21:02:12
tags:
  - Docker
  - DevOps
category:
  - 【Docker相关】
---

Docker容器化技术已经越来越成为环境部署的主流趋势，一键式部署，并且保证线上线下环境的一致性，节省问题排查的时间，维护效率高，并且对底层硬件的要求比传统虚拟机低，可以提升系统资源的利用率，开发 Docker容器技术所实现的微服务架构，即：将一个环境的各个部分（nginx网络服务、web网站代码数据、mysql/redis/elasticsearch等数据库服务）进行分离，再通过网络架构实现网络通信。

<!--more-->

## Vagrant基本使用

`vagrant --help`

指定虚拟机的目录：
`cd ~/Vagrant/centos7/`
`vagrant init centos/7`
创建一个 centos 7 的虚拟机，在当前目录初始化一个 Vagrantfile 文件，
描述了需要创建的虚拟机
config.vm.box = "centos/7"

`vagrant up`
启动 centos 7 的 VM，如果有的话直接加载，如果没有的话要去网上下载
，启动完成之后，就可以在 virtualbox 中看到成功创建了一个 centos7 的虚拟机

然后在 centos7 目录下执行：
`vagrant ssh`
进入到 centos7 的 VM 中

查看当前已经有的虚拟机：
`vagrant status`

停掉运行的 VM
`vagrant halt`

删除创建的 VM
`vagrant destroy`

通过 Vagrantfile 可以创建各种虚拟机，
可以从 vagrant cloud 网上找相应的 Vagrantfile 来创建相应环境的虚拟机

在 Vagrantfile 中配置 provision,使得启动虚拟机时可以自动运行相关命令

示例，配置 Vagrantfile 文件，使得启动虚拟机时自动安装配置好环境：

```bash
config.vm.provision "shell", inline: <<-SHELL
    sudo yum update
    sudo yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-selinux \
                docker-engine-selinux \
                docker-engine
    sudo yum install -y yum-utils \
                device-mapper-persistent-data \
                lvm2
    sudo yum-config-manager \
                --add-repo \
                https://download.docker.com/linux/centos/docker-ce.repo
    sudo yum install -y docker-ce
    sudo groupadd docker
    sudo gpasswd -a docker vagrant
    sudo service docker restart
SHELL
```

## Docker Machine

### 1、在虚拟机中自动安装 Docker Engine 的工具

```bash
#创建一个安装好docker的linux虚拟机，命名为demo
docker-machine create {demo}

#查看当前创建的虚拟机和相应的状态
docker-machine ls

#进入到相应的虚拟机
docker-machine ssh {demo}
```

### 2、docker-machine 在 mac 中的应用：

```bash
#查看mac上的docker信息
docker version

#将mac的服务器端连上demo的里的docker服务器端,实现远程管理docker machine
docker-machine env {demo}
eval $(docker-machine env {demo})
#此时在mac上运行docker version查看的server端的信息就是demo这台虚拟机中的docker server信息

#
docker-machine env --unset
eval $(docker-machine env --unset)
```

### 3、docker machine 在阿里云上的使用

docker machine 可以通过这种方式实现本地 docker 命令远程管理 docker machine

在本地运行 docker 命令创建远程服务器上的 docker 容器：
使用 driver（在官网上找第三方 driver: 阿里云`docker-machine-driver-aliyunecs`）

安装目录：`/usr/local/bin/docker-machine-driver-aliyunecs`

验证阿里云 ecs driver 安装成功
`docker-machine create -d aliyunecs --help`

#### 使用

- 前提：账户余额大于 100

- 步骤：

（1）阿里云控制台： 创建用户的 access key(访问控制=》用户管理=》创建用户=》创建 AccessKey)

（2）本地命令行，创建阿里云的 docker host：

```bash
#执行的命令：
`docker-machine create -d aliyunecs --aliyunecs-io-optimized=optimized --aliyunecs-instance-type=ecs.c5.large --aliyunecs-access-key-id=L{access-key-id} --aliyunecs-access-key-secret={access-key-secret} --aliyunecs-region={region} {name}
```

参数说明：

`--aliyunecs-io-optimized=optimized` io 优化的参数，必须要添加
`--aliyunecs-instance-type=ecs.c5.large` 实例的类型
`--aliyunecs-region={region}` 创建的实例所属地域
`{imooc}` 创建的实例名称

（3）本地命令行，连接阿里云的 docker server

```bash
docker-machine env {imooc}
eval $(docker-machine env {imooc})
```

创建完成之后就可以在阿里云的控制台中看到刚刚创建的实例。但是不用的时候及时删除，以免产生不必要的收费。

## Docker 架构和底层技术简介

## 镜像 Image

### 1.什么是 Image

- 文件和 meta data 的集合（root filesystem）
- 分层的，并且每一层都可以添加、改变、删除文件，成为一个新的 image
- 不同的 image 可以共享相同的 layer
- Image 本身是 read-only 的

### 2.Image 的获取

（1）自行编译创建

```bash
#1.创建Dockerfile
FROM ubuntu:14.04
LABEL maintainer="zqunor@gmail.com"
RUN apt-get update && aput-get install -y redis-server
EXPOSE 6379
#启动redis-server
ENTRYPOINT [ "/usr/bin/redis-server" ]

#2.build镜像 .表示根据当前目录下的Dockerfile进行build
docker built -t zqunor/redis:latest .
#5行Dockerfile，对应5步build过程，5步对应5层
```

（2）从 Docker 仓库中拉取

```bash
docker pull ubuntu:18.04
```

### 3.创建 Image

每一个 base image 都是一个可执行文件，执行 image 就是创建一个容器

（1）创建 docker 用户组和用户，避免每次使用 docker 命令都需要加 sudo

```bash
#创建docker用户组
sudo groupadd docker

#将当前用户家到docker的用户组中，使当前用户有使用docker的权限
sudo gpasswd -a vagrant docker

#创建完成之后，重启docker服务
sudo service docker restart
```

（2）Image 体验: hello-world

```bash
#获取镜像
docker pull hello-world
#查看镜像的基本信息
docker image ls
#执行image,创建容器
docker hello-world
```

（3）制作 Image

```bash
#准备工作：创建一个单独的目录，编写.c文件，并执行生成可执行文件，将这个可执行文件
vim hello.c
#include<stdio.h>
int main()
{
    printf("hello docker\n");
}
#配置c语言执行环境
sudo yum install gcc glibc-static
#执行生成可执行程序hello
gcc -static hello.c -o hello
#生成可执行文件以后，就可以在当前目录下创建Dockerfile文件
FROM scratch
ADD hello /
CMD ["/hello"]
#build成image
docker build -t zqunor/hello-world .
#查看image层级
docker history {image_id}
#执行image
docker run zqunor/hello-world
```

## 容器 Container

### 1.什么是 Container

- 通过 Image 创建（copy）
- 在 Image layer 之上建立一个 container layer(可读写)
- 类比面向对象：类和实例
- Image 负责 app 的存储和分发，Container 负责运行 app

### 2.相关命令

```bash
# 查看当前所有的容器，运行中和退出的
docker container ls -a = docker ps -a
#查看正在运行的所有容器
docker containrt ls = docker ps
#删除一个容器
docker container rm {container_id} = docker rm {container_id}
#删除一个镜像
docker image rm {image_id} = docker rmi {image_id}
#列出所有的container id
docker container ls -aq = docke container ls -a | awk {'print$1'}
#删除所有的container
docker rm $(docker container ls -aq)
#删除所有退出的container
docker rm $(docker container ls -f "status=existed" -q) = docker container prune
```

## 构建 Docker 镜像

(1)建立在已有的 image 上，创建 container, 然后在 container 中执行操作，操作完后，创建一个新的 image【不推荐，不安全】

```bash
docker commit {container_name} {repository:tag}
```

这样生成的 image 通过`docker history {container_id}`可以看到有相同的历史版本信息

(2)通过 Dockerfile 中的配置命令进行构建【推荐，安全】

```bash
> cd docker-centos-vim
> vim Dockerfile

FROM centos
RUN yum install vim

> docker build -t zqunor/centos-vim-new .
```

分析：

Dockerfile 构建镜像过程中，执行安装操作时对 image 的写操作，而 image 本身是只读的，所以在执行安装 vim 之前，先创建了临时的 container, 然后在 container 中执行安装 vim 的命令，之后再删除临时的 container，安装完成后再 commit 一个新的镜像。

## Dockerfile 语法梳理

### 关键字

(1) FROM image 来源

```bash
FROM scratch  # 制作base image

FROM centos # 使用base image
```

尽量使用官方的 image 作为 base image，安全

(2) LABEL 标签备注记录信息

```bash
LABEL maintaner="xiaoquwl@gmail.com"
LABEL verion="1.0"
LABEL description="This is description"
```

备注 image 的基本信息。

(3) RUN 执行命令,并创建新的 Image Layer

```bash
RUN yum update && yum install -y vim \
    python-dev # 反斜线换行
```

执行命令，每 run 一次，就会产生一个分层，为了避免产生无用分层，应该合并成一个 run，负责的 RUN 使用反斜线"\"换行

(4) WORKDIR 跳转或创建目录

```bash
WORKDIE /test
WORKDIR tmp
RUN pwd
# 输出/test/tmp
```

进入到相应目录下，没有的话就创建该目录。

尽量使用绝对路径，并且不要用`RUN cd`跳转目录

(5) ADD and COPY 拷贝

```bash
WORKDIR /root
ADD hello test/
# /root/test/hello

WORKDIR /root
COPY hello test/
# ?? /root/test/hello
```

- ADD|COPY：把本地的文件添加到 docker 中

- ADD：

  - 如果是压缩文件，会添加到 docker 并解压；
  - 如果添加的目录不存在，会创建该目录

- COPY

  - 大部分情况，COPY 由于 ADD

- 添加远程文件/目录要使用 curl 或者 wget

(6) ENV 设置常量，环境变量

```bash
ENV MYSQL_VERSION 5.6
RUN apt-get install -y mysql-server= "${MYSQL_VERSION}" \
    && rm -rf /var/lib/apt/lists/*
# 引用常量，在RUN中可以直接使用
```

(7) VOLUME and EXPOSE 存储和网络

(8) CMD and ENTRYPOINT

- CMD：设置容器启动后默认执行的命令和参数
  - 容器启动时默认执行的命令
  - 如果 docker run 指定了其他命令，CMD 命令被忽视
  - 如果定义了多个 CMD，只有最后一个会执行

- ENTRYPOINT：设置容器启动时运行的命令
  - 让容器以应用程序或者服务的形式运行
  - 不会被忽略，一定会执行

### Dockerfile 格式： Shell 格式和 Exec 格式

(1) 基本样式

- Shell 格式

```bash
RUN apt-get install -y vim
CMD echo "hello docker"
ENTRYPOINT echo "hello docker"
#输出 hello docker
```

- Exec 格式

```bash
RUN [ "apt-get", "install", "-y", "vim" ]
CMD [ "/bin/echo", "hello docker" ]
ENTRYPOINT [ "/bin/bash", "-c", "echo hello $name" ]
#输出 hello docker

ENTRYPOINT ["/bin/echo", "hello $name"]
#输出hello $name [echo指令未被识别]
```

(2) shell 格式示例

```bash
FROM centos
ENV name Docker
ENTRYPOINT echo "hello ${name}"
```

## Dockerfile实践

### 将python程序打包成image，并运行成为container

准备：
1.python环境
2.flask框架

编写Dockerfile

```bash
FROM python:2.7
LABEL  maintainer="zqunor<zqunor@gmail.com>"
RUN pip install flask
COPY app.py /app
WORKDIE /app
CMD ["python", "app.py"]
```

```python
from flask import flask
app = Flask(__name__)
@app.route('/')
def hello():
    return 'hello docker"
if __name__ == '__main__':
    app.run();
```


## Dockerfile实战2

```bash
#进入到ubuntu容器中
docker run -it ubuntu
#安装stress压力测试工具
apt-get update && apt-get install -y stress
#查看stress安装目录
which stress
#查看stress命令帮助信息
stress --help
#启动一个进程，并输出debug信息（默认为进程分配256MB）
stress -vm 1 --verbose
```


## 遇到的问题

### 1.编辑 Vagrantfile，设置启动虚拟机时自动执行的命令，但是进入到虚拟机中后，docker 没有启动，手动启动 docker 后，有报错:

`Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.39/version: dial unix /var/run/docker.sock: connect: permission denied`

=> 1.需要将当前用户加入到docker用户组，或者在执行的docker命令前加sudo

未完待续....