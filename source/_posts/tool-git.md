---
title: 使用Git和Github进行代码管理
date: 2018-05-16 13:47:52
tags:
    - git
    - github
category:
    - Tool
    - Git
---

**1、安装 Git**

官网下载地址：[下载]

学习教程:

* 官方手册：[前往]

* Pro Git: [查看]

**2、生成 ssh 秘钥**

`ssh-keygen`

中间出现提示进行设置 ssh 秘钥的存放地址，此处可直接回车，放到默认的存储位置`/c/Users/Administrator/.ssh/`
<!--more-->
![](https://images2018.cnblogs.com/blog/1049028/201803/1049028-20180318212746038-1187254415.png)

**3、查看公钥信息**

`cat /c/Users/Administrator/.ssh/id_rsa.pub`

![](https://images2018.cnblogs.com/blog/1049028/201803/1049028-20180318213326726-1484496635.png)

**4、放到 github 网站上**（设置秘钥入口：[传送门]）

**5、测试秘钥是否能够成功访问 github 网站**

`ssh -T git@github.com`
中间需要手动输入进行确认

![](https://images2018.cnblogs.com/blog/1049028/201803/1049028-20180318213631372-1010547178.png)

**6、成功连接，后续即可直接进行项目管理**

[下载]: https://git-scm.com/downloads
[前往]: https://git-scm.com/docs
[查看]: https://git-scm.com/book/zh/v2
[传送门]: https://github.com/settings/keys
