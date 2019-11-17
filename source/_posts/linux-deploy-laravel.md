---
title: CentOS上线部署laravel项目
date: 2019-05-13 23:42:07
tags:
  - laravel
category:
  - 【Linux相关】
---

闲置的服务器和域名，以后每天尽量抽出时间学习心得东西，并动手做一些实际的“产品”出来，为了今后自己职业生涯的进一步发展，服务器上一直都只是使用，或者跑起来一个项目就完事了，也没有怎么好好维护好，所以现在慢慢捡起来，开始积累了。

<!--more-->

### 安装lnmp环境

参考：
[简书 - Centos 7 下安装LNMP官方最新版](https://www.jianshu.com/p/1e51985b46dd)

### 安装redis

参考：[简书 - Centos 7下使用yum安装redis](https://www.jianshu.com/p/ebda253a8daa)

### 安装nodejs npm

> nodejs分8.x和10.x，这里用10.x的，如果需要用8.x，就换成`setup_8.x`

`curl -sL https://rpm.nodesource.com/setup_10.x | sudo bash -`

`yum -y install nodejs npm`

参考：[Linuxhint - CentOS安装nodejs、npm](https://linuxhint.com/install_nodejs_centos7/)

### 服务器运行Laravel

创建.env文件，并配置相关参数

1、创建`.env`文件（从模板`.env.example`文件复制来）

```bash
cp .env.example .env
```

2、配置参数

（1）默认文件中是没有app_key的，需要手动生成`APP_KEY`

```bash
php artisan key:generate
```

（2）生成后访问网站

![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g2zus87z2oj30nh0kodgj.jpg)

## 遇到的问题

【1】、修改了虚拟主机配置文件后，重启nginx`service nginx restart`的时候，报错：

```
Redirecting to /bin/systemctl restart nginx.service
Job for nginx.service failed because the control process exited with error code. See "systemctl status nginx.service" and "journalctl -xe" for details.
```

**原因:** 配置文件末尾少了';'

**操作**：在缺少分号结尾的行尾加上分号

**参考：**
[DigitalOcean - Can't start Nginx - Job for nginx.service failed](https://www.digitalocean.com/community/questions/can-t-start-nginx-job-for-nginx-service-failed)


【2】、执行`yum install`时，返回`yum except KeyboardInterrupt, e`错误

**原因：** `/usr/bin/yum`执行程序是使用python命令进行执行的，由于之前操作，将python默认的2.7版本替换成了3.7，导致`yum`程序的语法不支持，继而报错。

**参考:**  [CSDN - yum except KeyboardInterrupt, e: 错误](https://blog.csdn.net/Harith/article/details/17691137)

【3】、加载好laravel项目后，执行`composer install`后报`The Process class relies on proc_open, which is not available on your PHP installation.`

**原因**：`php.ini`文件配置中，禁用了`proc_open`、`proc_get_status`等方法。

**解决**：vim /usr/local/php72/etc/php.ini，找到`disable_functions`，然后删除`proc_open`、`proc_get_status`，然后再执行`composer install`。（如果执行后还有其他输出信息说方法disabled，同样的在相应位置删除即可）

**参考**：[CSDN - Laravel安装报错](https://blog.csdn.net/dahuzix/article/details/73377622)

## 参考资料

1、[OneinStack:   LNMP环境镜像使用手册](https://oneinstack.com/docs/lnmpstack-image-guide/)

2、[Github:   线上服务器自动部署的webhooks](https://developer.github.com/webhooks/)


3、[凹凸实验室:  使用Github的webhooks进行网站自动化部署](https://aotu.io/notes/2016/01/07/auto-deploy-website-by-webhooks-of-github/index.html)

4、[Linuxhint: CentOS安装nodejs、npm](https://linuxhint.com/install_nodejs_centos7/)
