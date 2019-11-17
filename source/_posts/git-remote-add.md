---
title: 【Git】将现有项目提交到空仓库
date: 2018-04-16 13:57:50
tags:
  - git
category:
  - 【开发工具】
---

如果想把本地的一个项目进行托管，应该如何操作？如何将本地的项目和远程的仓库进行连接管理？

关键点：`git remote add origin 远程地址名`

<!--more-->

### 本地项目执行操作

1.在本地项目目录下初始化 git 仓库

`git init`

2.将本地项目下工作区的所有文件添加到 git 版本库的暂存区中

`git add .`

（可以创建.gitignore 文件忽略不需要加入到版本库中的文件，或单独 git add {filename}将文件加入到版本库）

3.将暂存区的文件进行提交到版本库

`git commit -m '{描述}'`

### 远程 github 执行操作

创建一个仓库(仓库名任意)，并复制仓库地址`git@github.com:zqunor/lamp.git`

### 设置本地项目版本库的远程仓库地址

两种方式：

(1)使用 ssh 方式：

`git remote add origin git@github.com:zqunor/lamp.git`

(2)使用 http 方式：

`git remote add origin https://github.com/zqunor/lamp.git`

> 区别: 是 ssh 方式当把本地的`ssh key`公钥放到 github 上后就可以直接使用 push 和 pull 等操作，而 http 方式需要手动输入 github 账号的用户名和密码，进行验证

### 将本地版本库推送到 github 上

`git push origin master`

完成同步
