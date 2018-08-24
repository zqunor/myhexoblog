---
title: 【Docker】更新docker镜像源
date: 2018-07-31 10:00:00
tags:
  - 镜像
category:
  - Docker
---

使用 docker 拉取 Docker Hub 上镜像时，可能会由于网络限制，导致下载失败。可以将 docker 的镜像源设置为国内的镜像，
目前支持的镜像源有[阿里云](https://cr.console.aliyun.com/cn-hangzhou/mirrors)和[docker 中文站](https://www.docker-cn.com)

<!--more-->

1.获取个人加速地址：

[阿里云镜像加速器](https://cr.console.aliyun.com/cn-hangzhou/mirrors)

2.配置镜像加速器

> 针对 Docker 客户端版本大于 1.10.0 的用户

修改 daemon 配置文件`/etc/docker/daemon.json`来使用加速器

(1)创建`/etc/docker`目录(如果不存在的话)

`sudo mkdir -p /etc/docker`

(2)修改配置文件`/etc/docker/daemon.json`

[阿里云]：

```json
// 文件位置: /etc/docker/daemon.json
{
  "registry-mirrors": ["https://y0qd3iq.mirror.aliyuncs.com"]
}
```

[docker 中文站]：

```json
// 文件位置: /etc/docker/daemon.json
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

3.重载守护进程文件，重启 docker

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

4.查看加速器是否生效

```bash
[root@txyun ~]# docker info

...
Registry Mirrors:
 https://y0qd36iq.mirror.aliyuncs.com
...
```
