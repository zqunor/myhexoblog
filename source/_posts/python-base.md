---
title: 【Python】安装与配置
date: 2018-07-30 20:02:53
tags:
  - python
category:
  - 【Python相关】
---

拓展自己对语言的认识和理解，开始学点其他的语言，不给自己的技术栈设限，有更多的思考和认识。学习python, 了解些机器学习，人工智能方面的知识，希望自己可以坚持深入学习。

<!--more-->

## 下载安装

1、下载源码包：

```bash
wget https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tgz
```

2、解压缩：

```bash
tar -zxvf Python-3.7.0.tgz
```

然后进入到解压缩后的目录`cd Python-3.7.0`

3、安装配置：

```bash
./configure --prefix=/usr/local/python37/
```

> --prefix 是指定安装目录

> 执行前确保系统已安装了编译器gcc `sudo yum install -y make gcc gcc-c++`

4、编译，并安装：
```bash
make && make install
```

## 将默认的python2.7换成新版python3.7

(1) 查看当前系统python执行程序的位置

```bash
locate */bin/python
```

定位到加载的python目录是`/usr/bin/python`

(2) 查看环境变量的值`$PATH`

```bash
echo $PATH
```

有一行是`/usr/bin`, 说明直接运行的是`/usr/bin/python`， 并且当前`/usr/bin/python`是软连接到`/usr/bin/python2.7`的

(3) 建立新的软连接

使`/usr/bin/python`指向`/usr/local/python37/bin/python3.7`

```bash
ln -sb /usr/local/python37/bin/python3.7 /usr/bin/python
```

> -s: 表示建立的是软连接（快捷方式指向），不加表示是硬连接（即复制一份的）
> -b：表示删除、覆盖以前建立的链接

参考: [博客园 -- 每天一个linux命令（35）：ln 命令](https://www.cnblogs.com/peida/archive/2012/12/11/2812294.html)


此时执行python即是python3.7版本
![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g2ynt37gvzj30i102vmx4.jpg)

## 操作遇到的问题：

1、make install安装python3.7时报 No module named '_ctypes' 错误的解决办法：

答：`yum install libffi-devel`

参考： [StackOverflow -- Python3: ImportError: No module named '_ctypes' when using Value from module multiprocessing](https://stackoverflow.com/questions/27022373/python3-importerror-no-module-named-ctypes-when-using-value-from-module-mul)
