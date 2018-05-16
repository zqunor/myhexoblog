---
title: Composer设置国内镜像
date: 2018-05-16 13:20:37
tags:
    - composer
    - 镜像
category:
    - Tool
    - Composer
---

使用 composer 时，输入命令执行后半天没有反应，并最后是失败的消息。如下载项目中的框架文件时：
`composer install`
一直没有反应

<!--more-->

【注】添加参数`-vvv`可尽可能多的输出执行信息，帮助查看问题所在。

如：使用`composer`安装项目的框架文件时，等待时间过长，且没有其它输出。

可使用`-vvv`参数输出详细信息:
`composer install -vvv`

![](https://images2018.cnblogs.com/blog/1049028/201804/1049028-20180415234535780-570407396.png)
此时可以发现在做网络请求时出现的长时间等待，于是可以猜测是国内的网络限制的问题。

**解决办法**：

设置国内镜像：[官方介绍][官方介绍]

1、系统全局配置

`composer config -g repo.packagist composer https://packagist.phpcomposer.com`

2、单个项目配置

进入项目目录，执行命令

`composer config repo.packagist composer https://packagist.phpcomposer.com`

设置好镜像以后便可成功执行

![](https://images2018.cnblogs.com/blog/1049028/201804/1049028-20180415234950392-1812401653.png)

[官方介绍]: https://pkg.phpcomposer.com/
