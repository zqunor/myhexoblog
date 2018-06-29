---
title: [转]windows系统下的高效工具
date: 2018-06-29 18:50:47
tags:
    - tool
category:
    - Tool
---

正所谓：工欲善其事，必先利其器
花点时间来折腾一下还是非常有必要。

Mac 之所以 **高效**，实际上是藉其自带特色功能以及原生的命令行支持
（当然也有它脑残的一面，例如：自带的窗口管理基本没带；新建文件只能使用命令 ```touch``` ······）

<!--more-->

# ※ 高效特性

下面列出 Mac 相对高效的特性：

- 多工作区支持（实际上 Ubuntu 等都有此功能）
- 原生支持 Unix like 命令环境，且自带众多的运行环境及命令行工具
- 强大的 Spotlight 全局搜索框
- 强大的第三方神级软件：Homebrew / Alfred / iTerm2 ...
- ······

我们来把上述特性一一加装在 Windows 上。

P.S 首选 免费 / 开源 的，且在同类软件中脱颖而出的

# ※ 实现方案

## 多工作区（多桌面）支持：Dexpot

没有 Mac 的好用，但绝对够用了
商业软件推荐：AltDesk（不推荐使用 MS 自家的）

## 命令行环境：Babun（Cygwin + Oh-my-zsh + git + 众多运行环境）

Babun 自带包管理工具 pact，极其方便
[P.S] Babun 使用 Ctrl + C 中断 node 进程，实际上 node 依旧还在运行，见 issue

因此最终建议使用超级好用的 Git Bash

## 全局搜索框 / 快速启动器：Wox（Alfred for Windows，开源）

  而且还捎带上文件搜索神器 `Everything`，因此除了没有工作流外，基本满足开发需求。
  该项的选择非常多， `Launchy`、`ALTRun`、`FARR`，以及大神们最喜欢用的 win + R

## 其余效率相关

- 护眼神器：f.lux。不用多介绍了吧

- 增强型文件管理器：Total Commander 及其 教程。不过我用的是稍微简单一点的 Directory Opus

- 触发角与手势：WGestures

可媲美 Mac 的体验，甚至还有所加强！设置滑到边角就锁屏，懒得按键盘 win + L 了有木有！还可以配置写命令，国产开源的良心之作！快点去人家的 Github 上面点个星星吧亲！

- 历史剪贴板：**Ditto**，媲美 Mac 上的 ClipMenu，但又有所增强（带历史搜索功能）

- 简约又强大的卸载器：GeekUninstaller，可让你避免全家桶套餐

WAMP 集成环境：Mac 下自带 Apache 与 PHP，但我相信 WampServer 的使用会更方便

快捷键：实际上这才是最重要的，详请知乎

转自：![github](https://github.com/kenberkeley/make-windows-development-mac-like)
