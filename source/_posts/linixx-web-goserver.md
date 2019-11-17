---
title: 【Linux】网站上线
date: 2018-07-7 12:00:00
tags:
  - 上线
category:
  - 【Linux相关】
---

作为后端程序员，对网站上线的操作需要有一定的了解，对于一些没有专门运维人员的公司，运维上线的操作就需要后端程序员来执行。
需要学习域名解析、本地文件和服务器文件传递`scp`的相关操作。

<!--more-->

## 一、上传网站到服务器

### 1.将本地文件上传到远程服务器：**scp**

#### (1) 具体用法：

> scp -r {本地目录的文件} {服务器用户名@远程服务器 ip:{远程服务器的文件目录}}

`scp -r ./demo/* root@47.94.255.230:/root/www`

### 2.CentOS 系统管理命令：

```bash
# 使用ssh登录远程服务器
ssh {user}@{ipv4}

# 安装scp命令
yum install openssh-client

# 查看进程
ps -ef | grep nginx

# 验证nginx配置文件语法
nginx -t

# 关闭nginx服务进程
nginx -s stop

# 开启nginx服务进程
nginx -s reload
```

## 二、域名解析

### 1.解析设置

| 记录类型 | 主机记录 | 解析线路(jsp) | 记录值 | TTL 值  |
| :------: | :------: | :-----------: | :----: | :-----: |
|    A     |   www    |     默认      | {ipv4} | 10 分钟 |
|    A     |    @     |     默认      | {ipv4} | 10 分钟 |

### 2.说明

(1) `www`表示对有`www`前缀的完整域名进行解析

(2) `@`表示对没有`www`前缀的省略域名进行解析

(3) `CNAME`表示需要将域名重定向到另一个域名[使用 github 搭建博客绑定独立域名时需要用到]

## 三、HTTP 协议在访问域名时的工作流程

```ini
输入网址(imooc.com)
        ||
        \/
DNS解析，寻找对应服务器地址
        ||
        \/
进行第一次握手（HTTP会话）
        ||
        \/
建立文档树，加载资源
```