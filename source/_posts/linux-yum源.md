---
title: 【CentOS】更新yum软件源
date: 2018-07-31 9:00:00
tags:
  - 镜像
category:
  - 【Linux相关】
---

当使用 linux 安装包命令安装相关软件时，会提示`Connection timed out`,表示连接超时，即访问国外网址受限，可以切换软件源地址为国内的。

<!--more-->

目前有网易和阿里云：

- [网易软件源](mirrors.163.com)

- [阿里云软件源](https://opsx.alibaba.com/mirror)

【以阿里云为例(CentOS)】：

(1) 备份

```bash
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

(2) 下载新的 CentOS-Base.repo 到`/etc/yum.repos.d/`

[CentOS 5]

```bash
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-5.repo
# 或
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-5.repo
```

[CentOS 6]

```bash
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
# 或
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
```

[CentOS 7]

```bash
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# 或
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

(3) 生成缓存，是 yum 源生效

```bash
yum clean all
yum makecache
```

(4)更新 yum,验证 yum 软件源

```bash
[root@centos ~]# yum -y update
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
No packages marked for update
```
