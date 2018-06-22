---
title: 【VSCode插件】xdebug开发调试PHP
date: 2018-05-09 16:09:38
tags:
    - tool
    - vscoode
    - xdebug
category:
    - Tool
    - VSCode
toc: true
---

Xdebug 在开发过程中可以帮我们查看具体的运行和步骤，以及每行代码执行的结果，在学习和解决代码问题的时候可以提供非常大的便利。PHPStorm 也可以进行 Xdebug 调试，VScode 也可以进行配置调试，且比 PHPStorm 的配置简单很多，不用每次去创建一个 Server，再创建一个 web page 服务。相比之下，VSCode 的界面好看，且简单方便，值得学习一下。

使用了一段时间，但是偶尔还是会出现一些问题，故而进行了整理总结。

<!-- more -->

## 一.插件准备

### 1.查看插件列表

[[avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/vscode-debug-01.png)

### 2.搜索并安装`PHP Debug` (安装 VScode 时选择 PHP 开发相关的话会自动安装)

**PHP Debug**
![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/vscode-debug-02.png)

## 二.进行配置

### 1.给 PHP 安装 Xdebug 扩展(此处使用的是 PHPstudy 集成开发环境)

![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/vscode-debug-04xdebug.png)

### 2.在 php.ini 中添加相关配置

```ini
[XDebug]
# xdebug扩展的位置，phpstudy已经默认设置好
zend_extension="D:\phpStudy\PHPTutorial\php\php-5.6.27-nts\ext\php_xdebug.dll"
xdebug.auto_trace=1
xdebug.collect_params=1
xdebug.collect_return=1
xdebug.trace_output_dir ="D\phpStudy\tmp\xdebug"
xdebug.profiler_output_dir ="D:\phpStudy\tmp\xdebug"
xdebug.profiler_output_name = "cachegrind.out.%t.%p"
xdebug.remote_enable = 1
xdebug.remote_autostart = 1
xdebug.remote_handler = "dbgp"
xdebug.remote_host = "127.0.0.1"
# 设置端口号，默认是9000，此处因为本地环境端口冲突故设置为9001（在vscode配置中需要用到）
xdebug.remote_port = 9001
# 这是用于phpstorm中xdebug调试的配置，在vscode中没有用到
xdebug.idekey = phpstorm
```

### 3.在 phpinfo 中查看 xdebug 扩展的信息，验证是否开启成功

![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/phpstrom_xdebug/08.png)

![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/phpstrom_xdebug/09.png)

![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/phpstrom_xdebug/10.png)

### 4.查看 vscode 中 debug 页面

![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/vscode-debug-03.png)

### 5.新建 debug 配置,并选择调试语言

![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/vscode-debug-05choose.png)

### 6.进行配置

![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/vscode-debug-05setting.png)
相关配置信息参考：（注意 port 端口号的值，需要与 php.ini 中设置的一样）

```json
{
  // 使用 IntelliSense 了解相关属性。
  // 悬停以查看现有属性的描述。
  // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Listen for XDebug",
      "type": "php",
      "request": "launch",
      "port": 9001
    },
    {
      "name": "Launch currently open script",
      "type": "php",
      "request": "launch",
      "program": "${file}",
      "cwd": "${fileDirname}",
      "port": 9001
    }
  ]
}
```

## 三.开启调试

### 1.启动 debug(点击绿色小箭头启动)

![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/vscode-debug-06start.png)

显示出`调试小窗口`

![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/vscode-debug-06startbanner.png)

### 2.开启自动附加(单击即可切换开关)

![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/vscode-debug-06startfooter.png)

### 3.设置断点(行号前点击即可出现红色小断点)

![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/vscode-debug-07duandian.png)

### 4.在浏览器中访问设置断点的程序

### 5.访问后会自动跳转到 VSCode，并显示出断点标记，并显示相关执行结果

![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/vscode-debug-07start.png)

### 6.在`调试小窗口`中进行单步调试或单步跳过等操作

## 注意

1.注意**自动附加**是否是开启状态

2.注意端口号是否冲突（点击下部玫红色状态栏的`Listen for XDebug`后，会弹出选择 debug 设置如果端口设置有问题的话，会在选择后弹出错误提示）

![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/vscode-debug-08port.png)

(设置小图标后打开调试控制台也可以显示相关错误提示，注意查看即可)
将`launch.json`的端口号修改未被占用的号，并且修改`php.ini`中 xdebug 的配置
