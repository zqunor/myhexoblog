---
title: 【Linux】系统学习Crontab定时任务
date: 2018-07-06 12:00:00
tags:
  - crontab
category:
  - 【Linux相关】
---

crontab 是一个用于设置周期性执行任务的工具。目前服务器端的运行环境大多数为 Linux，在日常的运营和维护中会有很多需要定期执行的操作，其中有些操作是可以机械的定期执行的操作，所以我们可以使用 crontab 定时服务来设置定时任务，从而减少手动操作的任务，帮助提高工作效率。

<!--more-->

## 一、cron 定时任务

### 1.安装 crond 服务和 crontab 工具

#### (1)相关命令

```bash
# 清除yum缓存
yum clean all

# 更新系统的安装包到最新版本
yum update

# 安装cron服务和crontab工具[-y表示yes,没有的话需要手动输入yes]
yum install -y cronie crontabs
```

#### (2)验证 crond 服务

```bash
# 检查cond服务是否安装及启动：
yum list cronie && systemctl status crond

# 检查crontab工具是否安装：
yum list crontabs && which crontab && crontab -l
```

## 二、crontab 架构

### 1.执行步骤

```bash
# 1.新建|编辑定时任务
crontab -e

# 2.查看定时任务列表[当前用户(root)保存的计划任务]
crontab -l
#或
cat /var/spool/cron/root

# 3.重启crond进程
systemctl restart crond

# 4.查看crond状态
systemctl status crond
```

### 2.contab 配置文件格式

```bash
*     *     *    *    *   [username]   my command
分    时    日   月   周   [执行用户名]  要运行的命令
0-59 0-23 1-31 1-12  0-6
```

```info
* 取值范围内的所有数字
/ 每
- 某个区间
, 几个数的集合
```

### 3.crontab 配置文件

#### (1)系统配置文件

`/etc/crontab`

#### (2)系统用户 crontab 配置文件保存目录[crontab -e 所编辑的文件]

```info
/var/spllo/cron/[目录]
root:/var/spppl/cron/root[文件]
user01:/var/spppl/cron/user01[文件]
```

```bash
# 创建用户
useradd user01
# 使用指定用户登陆系统
su - user01
```

### 4.Crontab 环境变量

#### (1)添加 PATH 到/etc/crontab

```bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/jdk1.8.0_111/bin
# 可以直接在定时任务文件使用java程序
```

#### (2)在执行具体任务前引入系统/用户环境变量

```bash
# [系统级别环境变量]
30 2 * * * source /etc/profile;sh /root/test.sh

# [用户级别环境变量]
30 2 * * * source ~/.bash_profile;sh /root/test.sh
```

#### (3)操作步骤

##### 1)添加环境变量

```ini
# /etc/profile 或者 /root/.bash_profile
PATH=$PATH:/usr/local/jdk1.8.0_111/bin
export PATH
```

##### 2)完成执行脚本

```ini
# /root/test.sh
java -version 2> /root/script.out
```

##### 3)设置定时任务

```bash
# 先使/etc/profile生效[添加环境变量]，后使用sh执行脚本文件
* * * * * source /etc/profile;sh /root/test.sh
* * * * * source /root/.bash_profile;sh /root/test.sh
```

## 三、实战

### 1.crontab 日志

#### (1)cron 日志保存在`/var/log/cron`文件中

```bash
# 查看最近的两条cron日志
tail -n 2 /var/log/cron
```

### 2.清理系统日志

#### (1)系统日志的存放位置： `/var/log/messages`

```info
# /var/log目录下日志[平时主要用到的日志文件]
cron -- 定时任务日志
secure -- 相关ssh服务日志
messages -- 系统总日志
firewalld -- 系统防火墙日志
lastlog -- 登录日志
```

```bash
# 查看当前目录下所有文件的大小
du -sh .

# 清空系统日志 [/dev/null代表一个空文件]
* 1 * * * cat /dev/null > /var/log/messages
```

#### (2)同时清理多个日志文件[使用脚本文件]

##### 1). 文件：`/root/log_clean.sh`

```shell
#!/bin/sh

cat /dev/null > /var/log/messages
cat /dev/null > /var/log/secure
```

##### 2). 定时任务

```bash
# 定时执行日志清理脚本
* * * * * sh /root/log_clean.sh
```

### 3.crontab 备份 source code

#### (1)准备工作

```info
backup -- 备份文件存放目录
script -- 脚本文件目录
www --- 网站根目录
```

#### (2)编写执行备份的脚本文件[/data/script/www_backup.sh]

```shell
#!/bin/bash

basedir = /data/backup

www_src = $basedir/www_src/$(date +%F_%H%M)

[ ! -d "$www_src" ]  && mkdir -p $www_src

cd /data
tar -jpcf $www_src/www.tar.bz2 www
```

#### (3)脚本解释说明

##### 1). `basedir = /data/backup`

定义一个变量，名称为`basedir`，值为`/data/backup`

##### 2). `$(date +%F_%H%M)`

获取系统的日期时间，并拼上日期(%F),再拼上小时(%H)和分钟(%M) =》 [2018-03-16_0715]

##### 3). `www_src = $basedir/www_src/$(date +%F_%H%M)`

使用变量 basedir 的值，再拼接出多层目录，赋值给 www_src[值为/data/backup/www_src/{当前日期时间的目录(动态)}]

##### 4). `[ ! -d "$www_src" ]`

相当于 if 的判断,判断$www_src 是否不是一个目录

##### 5). `tar -jpcf $www_src/www.tar.bz2 www`

打包/data/www 目录为 www.tar.bz2，并放置到$www_src 目录下[/data/backup/www_src/{当前日期时间的目录(动态)}/]

#### (4)创建定时任务

`* * * * * sh /data/script/www_backup.sh`

#### (5)重启定时任务使其生效

`systemctl restart crond`

#### (6)查看备份文件大小[被备份的原目录是 wordpress 系统]

```bash
# /data/backup/www_src/2018-03-16_0715/
8.4M    www.tar.bz2
```

### 4.crontab 在 iptables 上的应用

#### (1)应用场景：

DDOS 攻击或匿名暴力破解导致系统无法正常访问

#### (2)解决之道：

使用 crontab 集成 iptables，实时监控系统的网络状态，及时将可疑的 ip 地址加入到网络黑名单

#### (3)iptables 简介：

iptables 作为 Linux 下的内核防火墙，能够通过添加相应的规则，检测、修改、重定向、转发和丢弃 ip 数据包，从而过滤网络数据，实现保护系统网络的功能

CentOS7 默认使用 firewalld 服务维护内核防火墙，我们需要禁用 firewalld 服务，并安装 iptables 作为系统默认防火墙。

#### (4)准备工作

```bash
# 查看当前firewalld进程状态
systemctl status firewalld

# 关闭并禁用firewalld
systemctl disable firewalld
systemctl stop firewalld

# 安装iptables服务
yum install iptables-services
[yes]

# 开启iptables服务
systemctl enable iptables

# 启动iptables服务
systemctl start iptables

# 验证iptables是否安装成功
iptables -V
```

#### (5)设置网络访问黑名单文件[/data/script/blacklist.txt](每一行为一个ip地址)

#### (6)脚本实现[/data/script/firewall.sh]

```shell
#!/bin/bash

iptables -F

list = /data/script/blacklist.txt

for line in `cat $list`;do
  iptables -I INPUT -s$line -j DROP
  echo "$line is dropped into blacklist"
done
```

#### (7)脚本解释说明

##### 1). `iptables -F`

清空 iptables 的所有记录

##### 2). for line in `cat $list`;do

对/data/script/blacklist.txt 循环输出每一行

##### 3). `iptables -I INPUT -s$line -j DROP`

添加一个 input 链
-s 相当于 source，将每一行的数据，传递给 iptables
-j DROP 相当于一个 drop 操作，将当前主机接收到的数据包进行一个丢弃操作
=》将 blacklist.txt 中的 ip 地址做一个拒绝访问的操作，使黑名单中的 ip 地址无法访问主机

##### 4). `echo "$line is dropped into blacklist"`

对每一行的 ip 地址拒绝访问后，输出一条信息提示

#### (8)创建定时任务

`* * * * * source /etc/profile; sh /data/script/firewall.sh`

#### (9)查看 iptables 拒绝访问主机的 ip 列表

```bash
iptables -nvL
```

### 5.crontab 在 Jenkins 上的扩展

#### (1)Jenkins 介绍

    java编写的开源、持续、集成工具，最大优势就是将开发人员和运维人员完美的结合在一起。

#### (2)使用前提

```bash
# 安装
yum install jenkins
[y]

# 查看系统java路径
which java

# 查看系统java版本
java -version
```

> [jenkins 要求的最低版本要求配置>1.8.0_111]

#### (3)编辑 jenkins 的启动配置文件[/etc/init.d/jenkins]

[=>搜索 candidates 关键词]
默认调用的 java 目录是/usr/bin/java 和/usr/lib/jvm/\*\*\*/bin/java

添加 java 的路径
`/usr/local/jdk1.8.0_111/bin/java`

#### (4)相关命令

```bash
#启动jenkins服务
systemctl start jenkins

# 查看jenkins服务是否正常启动，并查看jenkins服务是否打开8080端口监听服务
lsof -i:8080
```

#### (5)在 web 网页中进行配置并使用 jenkins

```bash
# 1.访问jenkins的后台地址
10.110.16.5:8080

# 2.登录密码保存在系统的目录下
/var/lib/jenkins/secrets/initialAdminPassword文件中

# 3.自定义jenkins=》安装插件

# 4.创建第一个管理员用户

# 5.创建新任务
1)构建自由风格的软件项目
2)构建触发器 => Build Periodically => 日程表[相当于crontab定时任务的时间表单] => * * * * *
3)构建 => 增加构建步骤 => Execute shell => Command[执行定时任务时执行的操作] => echo "this is a test build"
```

【学习参考】：[慕课网-Crontab 不知疲倦的时间表](https://www.imooc.com/learn/1009)
