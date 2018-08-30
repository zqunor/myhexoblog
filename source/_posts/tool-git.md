---
title: 使用Git和Github进行代码管理
date: 2018-04-16 13:47:52
tags:
  - git
  - github
category:
  - Git
---

使用 Git 进行代码版本管理是程序员项目记录和管理的重要途径，并且为便于多设备能够共享代码，进行远程管理是一个比较理想的方式，而 Github 作为全球最大的开源代码管理社区也是非常好的远程仓库选择，并且其提供静态博客域名和服务器，可以直接将仓库作为博客内容源，对于没有购置服务器却希望搭建个人博客的程序员是非常友好的。

<!--more-->

## 安装 Git

官网下载地址：[下载](https://git-scm.com/downloads)

学习教程:

- 官方手册：[前往](https://git-scm.com/docs)

- Pro Git: [查看](https://git-scm.com/book/zh/v2)

## 生成 ssh 秘钥

`ssh-keygen`

中间出现提示进行设置 ssh 秘钥的存放地址，此处可直接回车，放到默认的存储位置`/c/Users/Administrator/.ssh/`

![](https://images2018.cnblogs.com/blog/1049028/201803/1049028-20180318212746038-1187254415.png)

## 查看公钥信息

`cat /c/Users/Administrator/.ssh/id_rsa.pub`

![](https://images2018.cnblogs.com/blog/1049028/201803/1049028-20180318213326726-1484496635.png)

## 放到 github 网站上

（设置秘钥入口：[传送门](https://github.com/settings/keys)）

## 测试秘钥是否能够成功访问 github 网站

`ssh -T git@github.com`
中间需要手动输入进行确认

![](https://images2018.cnblogs.com/blog/1049028/201803/1049028-20180318213631372-1010547178.png)

【问题调整】：

如果执行上述命令时，出现以下错误提示

```bash
Hi username! You've successfully authenticated, but GitHub does not
provide shell access.
```

则需要再对 ssh 配置文件进行配置`~/.ssh/config` [.ssh 的目录以自己安装时设置的目录为准]

```bash
Host github.com
Hostname ssh.github.com
Port 443
```

设置完成后再执行上述命令：

```bash
[root@VM_0_10_centos i2arch]# ssh -T git@github.com
The authenticity of host '[ssh.github.com]:443 ([192.xx.xx.123]:443)' can't be established.
RSA key fingerprint is SHA256xxxxxxxxxxxxxxxxxxxxxxxxLviKw6E5SY8.
RSA key fingerprint is MD5:1xxxxxxxxxxxxxxxxxxxxxxxxxxxxd:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[ssh.github.com]:443,[192.xx.xx.123]:443' (RSA) to the list of known hosts.
Hi zqunor! You've successfully authenticated, but GitHub does not provide shell access.
```

则证明已经可以使用 git 访问 github，后续即可直接进行项目管理

## 参考资料：

[Github Help](https://help.github.com/articles/using-ssh-over-the-https-port/)
