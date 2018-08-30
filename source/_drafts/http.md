---
title: 【HTTP】原理结合实践
date: 2018-07-5 12:00:00
tags:
  - http
category:
  - HTTP
---

<!--more-->

### HTTP 请求头信息

Cache-Control

- public、private：设置缓存仅在客户端缓存还是可以在代理服务器中缓存
- must-revalidate：客户端的缓存获取之后必须到服务端验证之后才能继续使用缓存
- no-cache、no-store：设置是否使用缓存

缓存验证

- last-modified 配合 if-modified-since
- etag 配合 if-none-match

- Contetn-Type 、Content-Encoding：等用来约束数据类型
- Cookie：保持会话信息
- CORS：实现跨域并保持安全性限制

## 深入到 TCP

1.什么是三次握手

2.HTTPS 链接的创建过程，以及为什么 HTTPS 是安全的

3.什么是长链接，为什么需要长链接

4.HTTP2 的信道复用为什么能提高性能

## HTTP 请求完整过程

![](https://ws1.sinaimg.cn/large/005EgYNMgy1fuqog8z9uij318n0f30xa.jpg)

1.Redirect 浏览器要对 url 判断是否需要重定向到新的 url 地址

2.缓存 判断是否需要到浏览器缓存中获取 url 对应的网页资源

3.DNS 解析 将域名解析成 ip 才能访问到资源所在的服务器

4.TCP 连接 拿到 ip 后就要进行 TCP 连接[ http: 经过 TCP 的三次握手;https: 不同于三次握手，需要另外的保证数据安全传输的过程]

5.Request 发送请求的数据包

6.Response 接收到请求数据包，并返回响应结果

## 网络协议分层

![](https://ws1.sinaimg.cn/large/005EgYNMgy1fuqoss50zpj30zv0pogpe.jpg)

- 物理层：定义物理设备如何传输数据
- 数据链路层：在通信的实体间建立数据链路连接
- 网络层：为数据在结点之间传输创建逻辑链路
- 传输层：
  - 向用户传输可靠的端到端(End-to-End)服务
  - 传输层向高层屏蔽了下层数据通信的细节
- 应用层
  - 为应用软件提供了很多服务
  - 构建于 TCP 协议之上
  - 屏蔽网络传输相关细节

## HTTP 发展历史

### 几个概念

1.http 并发数

2.http 请求

3.tcp 连接 [1 个连接可以有多个 http 请求]

### HTTP/0.9

- 只有一个命令(方法)GET
- 没有 HEADER 等描述数据的信息
- 服务器发送完毕，就关闭 TCP 连接

### HTTP/1.0

- 增加了很多命令
- 增加 status code(服务端处理完某个请求后的状态)和 header(发送和请求相关的数据 以及对这部分数据的操作方法))
- 多字符支持、多部分发送、权限、缓存等

[一个 http 请求就要建立一个 tcp 连接，返回完内容后就关闭，但是三次握手建立的消耗和延迟比较高]

### HTTP/1.1

- 增加持久连接 [建立完一个 tcp 连接后，在返回完数据后不关闭，让后续的所有请求都使用这个连接]
- pipeline [可以在同一个连接中发送多个请求，但是多个请求在服务器端需要顺序处理（串行），这样如果前面的请求处理的时间需要很长，而后面的请求处理的时间只需要很短，导致性能不太好]
- 增加 host(可以在同一台(物理)服务器跑多个 web 服务[node、java]，使用 host 进行判断，提高物理服务的使用效率)和其他一些命令

### HTTP/2.0

- 所有数据以二进制(帧)传输[以前是以字符串形式传输]
- 同一个连接里面发送多个请求不再需要需要按照顺序来（并行）
- 头信息压缩以及推送等提高效率的功能

## HTTP 三次握手

### HTTP 请求和 TCP 连接关系

![](https://ws1.sinaimg.cn/large/005EgYNMgy1fuqq6thfa2j30yy0eyjs6.jpg)

1.客户端和服务器之间的 http 请求的发送和返回的过程需要创建 TCP 连接

2.http 只有请求和响应的概念，没有连接的概念

3.请求和响应都是数据包，需要传输通道，TCP 连接就是数据包的传输通道（TCP 连接: 客户端发送，服务端接收）

### 三次握手时序图

![](https://ws1.sinaimg.cn/large/005EgYNMgy1fuqq70zeilj30tw0p4zmm.jpg)

1.第一次握手（SYN=1,Seq=X）

2.第二次握手（SYN=1,ACK=X+1,Seq=Y）

3.第三次握手（ACK=Y+1,Seq=Z）

网络抓包工具：wireshark

### URI URL URN

#### URI

【定义】： Uniform Resource Identifier 统一资源标识符

- 用来唯一标识互联网上的信息资源
- 包括 URL 和 URN

#### URL

【定义】：Uniform Resource Locator 统一资源定位器

示例： http://user:pass@host.com:80/path?query=string#hash

- **[ http ]**：传送协议。`Data URI scheme` [http ftp mailto]

- **[ // ]**：层级 URL 标记符号(为[//],固定不变)

- **[ user:pass ]**： 访问资源需要的凭证信息（可省略，不安全）

- **[ host.com ]**：服务器。（通常为域名，有时为 IP 地址）

- **[ 80 ]**：端口号。（以数字方式表示，若为 HTTP 的默认值“:80”可省略）

- **[ /path ]**：路径。（以“/”字符区别路径中的每一个目录名称）

- **[ ?query=string ]**：查询。（GET 模式的窗体参数，以“?”字符为起点，每个参数以“&”隔开，再以“=”分开参数名称与数据，通常以 UTF8 的 URL 编码，避开字符冲突的问题）

- **[ #hash ]**：片段。以“#”字符为起点。（锚点应用）

#### URN

【定义】：永久统一资源定位符

资源移动之后还能被找到

目前还没有成熟的使用方案

## 参考地址：

1. [wikipedia-统一资源定位符](https://zh.wikipedia.org/wiki/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E5%AE%9A%E4%BD%8D%E7%AC%A6)

未完待续