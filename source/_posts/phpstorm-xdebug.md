---
title: PHPStorm 配置 xdebug(phpStudy/wamp)
date: 2018-05-05 17:44:49
tags:
  - phpstrom
  - xdebug
category:
  - 【开发工具】
---

PHPStorm 是一款功能强大的 PHP 开发工具，自动补全、格式化样式等，以及最主要的 XDebug 功能，是开发中非常有用的功能，能有效查看程序代码的问题所在，并了解程序的执行过程。

<!-- more -->

## 一、下载并配置 XDebug

**wamp 环境**：
1、获取 xdebug
![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/phpstrom_xdebug/01.png)

官网地址：https://xdebug.org/wizard.php

**注**：需要将 phpinfo()输出的信息通过查看源码的方式将所有信息复制粘贴到 XDebug 的下载界面，以选择合适的版本进行下载和配置
![](https://ws1.sinaimg.cn/large/005EgYNMgy1fucho9kapjj30oi0gu40u.jpg)

将 phpinfo()的查看网页源代码的信息复制粘贴进后出现上述检测信息，然后进行下载，并按提示操作。

2、下载 dll 文件到扩展目录
![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/phpstrom_xdebug/03.png)

3、修改 php.ini 文件

（1）将 xdebug 文件引入
[avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/phpstrom_xdebug/04.png)

（2）开启 xdebug

![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/phpstrom_xdebug/05.png)
如果需要调试 Joomla 代码，则开启 XDebug profiling 。但是不用的情况下开启这个功能会降低系统稳定性，所以如果不是需要请勿开启。

（3）开启自动刷新

![](https://ws1.sinaimg.cn/large/005EgYNMgy1fuchoygxclj30ht056q39.jpg)

**phpStudy 环境**：
phpStudy 集成环境已经集成了 xdebug 扩展，只需开启即可。

1、开启方式：
![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/phpstrom_xdebug/07.png)
2、修改 php.ini 配置文件【XDebug 模块】

```ini
[XDebug]
zend_extension="D:\phpStudy\PHPTutorial\php\php-5.6.27-nts\ext\php_xdebug.dll"
;是否允许Xdebug跟踪函数调用，跟踪信息以文件形式存储，默认值为0
xdebug.auto_trace=1
;是否允许Xdebug跟踪函数参数，默认值为0
xdebug.collect_params=1
;是否允许Xdebug跟踪函数返回值，默认值为0
xdebug.collect_return=1
;函数调用跟踪信息输出文件目录，默认值为/tmp
xdebug.trace_output_dir ="D:\phpStudy\tmp\xdebug"
;性能分析文件的存放位置，默认值为/tmp
xdebug.profiler_output_dir ="D:\phpStudy\tmp\xdebug"
;打开xdebug的性能分析器，以文件形式存储，这项配置是不能以ini_set()函数配置的，默认值为0
xdebug.profiler_enable = 1
;性能分析文件的命名规则，默认值为cachegrind.out.%p
xdebug.profiler_output_name = "cachegrind.out.%t.%p"
xdebug.remote_enable = 1
;用于zend studio远程调试的应用层通信协议
xdebug.remote_handler = "dbgp"
xdebug.idekey = PHPSTORM
xdebug.remote_host = "127.0.0.1"
;
xdebug.remote_port = 9000
```

【注意】路径目录需要修改为自己对应的位置。

## 二、验证安装成功

1、修改配置后重启 apache 服务

2、在 phpinfo()的输出信息中查看 xdebug 信息
![](https://ws1.sinaimg.cn/large/005EgYNMgy1fuchppegacj30e802yjrl.jpg)

![](https://ws1.sinaimg.cn/large/005EgYNMgy1fuchq03aygj30oi03e74b.jpg)

![](https://ws1.sinaimg.cn/large/005EgYNMgy1fuchq788x8j30oi086ab2.jpg)

## 三、在 PHPStorm 中配置 xdebug

1、配置 PHP 版本信息
![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/phpstrom_xdebug/11.png)

2、设置 xdebug 端口（phpinfo()中显示默认 9000 端口）
![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/phpstrom_xdebug/12.png)

3、配置项目的服务器虚拟域名
![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/phpstrom_xdebug/13.png)

4、设置监听的域名和端口
![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/phpstrom_xdebug/14.png)

5、配置 xdebug

（1）进入配置

![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/phpstrom_xdebug/15.png)

（2）添加配置项，选择 PHP Web Page

![](https://ws1.sinaimg.cn/large/005EgYNMgy1fuchqnh71cj306k0dz3za.jpg)

（3）配置参数
![](https://ws1.sinaimg.cn/large/005EgYNMgy1fuchqxhn3oj30f80c974q.jpg)

## 四、安装浏览器插件（xdebug helper）

![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/phpstrom_xdebug/18.png)

## 五、在项目中使用 XDebug

1、开启浏览器中的 xdebug 插件

![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/phpstrom_xdebug/19.png)

2、在 PHPStorm 中进行监听

![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/phpstrom_xdebug/20.png)

3、在项目中设置断点标记
![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/phpstrom_xdebug/21.png)

4、在浏览器中访问项目

![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/phpstrom_xdebug/22.png)

5、运行后发现会在断电处停止
![](https://ws1.sinaimg.cn/large/005EgYNMgy1fuchrcso89j30bt00wa9u.jpg)

6、开始调试

F7 键：单步调试

Shift+F8：按区块调试

下方的调试面板会出现一下调试信息

![](https://ws1.sinaimg.cn/large/005EgYNMgy1fuchrnbc2fj30m508dgma.jpg)

## 六、完成

现在即可通过调试查看项目的运行步骤和文件跳转情况。

参考资料：查看[1] 查看[2]

[1]: https://segmentfault.com/a/1190000011907425?XDEBUG_SESSION_START=phpstorm
[2]: https://blog.csdn.net/zz_buddha/article/details/54096000
