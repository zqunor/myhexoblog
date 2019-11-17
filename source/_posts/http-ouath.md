---
title: 【HTTP】第三方登录OAuth2.0
date: 2018-07-5 12:00:00
tags:
  - oauth
category:
  - 【HTTP相关】
---

对于网站应用程序，涉及到登录和第三方 api 接口时，都会接触到 Token 等概念，而这部分的逻辑原理则是来自于 OAuth 授权协议，
目前的 OAuth2.0 协议的安全性也是被广泛认可，到目前为止尚且没有发生严重的安全事故。学习 OAuth2.0 协议的工作原理，并了解 qq 登录的流程和实现方式。

<!--more-->

## OAuth2.0 协议工作原理

![](http://ww1.sinaimg.cn/large/005EgYNMgy1fsvhp6rqkvj30wz0g0wld.jpg)

### 步骤一：请求 OAuth 登录页

Request Token URL - 未授权的令牌请求服务地址[慕课网请求 QQ 登录页面时使用的带有特定参数的 URL](client_id,redirect_uri)

### 步骤二：用户使用第三方账号登录并授权

身份认证通过后，会跳转到第一步的 redirect_uri，并携带 code 参数

### 步骤三：返回登录结果

User Authorization URL - 用户授权的令牌请求服务地址[用户 QQ 登录授权之后需要请求的一个带有特定参数的 URL](client_id,client_secret,code)
code 有生命周期且只可使用一次的字符串

AccessToken - 用户通过第三方应用访问 OAuth 接口的令牌[通过慕课网把自己喜欢的课程分享到 QQ 空间]
Refresh Token

### AccessToken 和 RefreshToken 数据传输原理

[imooc]带有 AccessToken 参数的特定 URL=>[post]=>[QQ]open Authorization API=>[xml/json]=>[imooc]带有 AccessToken 参数的特定 URL

### AccessToken 和 RefreshToken 生命周期解析

AccessToken - 具有较长生命周期(10 天半个月甚至更长)
User Authorization URL 中指定参数 RefreshToken 进行重新获取 AccessToken

## QQ 登录

### 1.接入 QQ 开放平台的前置条件

- qq 号
- 公网可访问的 web 服务器

- 关于域名备案

![](http://ww1.sinaimg.cn/large/005EgYNMgy1fsvhwnt4zqj30ye0fqdnz.jpg)

(腾讯的用于域名验证，拿到 appid 等信息)

- 关于服务器运行环境

### 2.申请 AppID 和 AppKey

[QQ 互联](https://connect.qq.com)

网站地址[需要在该页面下的 index.html 文件中嵌入一行代码，然后进行验证]
回调地址[可以填写多个，英文半角分号;间隔，加 http(s)://头]

[每次修改配置后都需要重新验证网站地址]

### 3.添加测试回调地址

eg. http://test.open.mypro.com/callback.php

### 4.引入官方 SDK

[下载](http://wiki.connect.qq.com/sdk%E4%B8%8B%E8%BD%BD)

### 5.SDK 参数配置

- appid
- appkey
- callback
- 请求授权列表

        get_user_info add_share list_album add_album ...

  [请求的权限会在授权登录页面显示需要请求的信息列表]

- 是否开通调试

### 6.SDK 解读

- 文档资料 -> oauth 开发指引 -> 开发功率\_server-side

- Server-side or Client-side

核心类和重要方法(Connectx.x/class/\*.class.php)

- Recorder.class.php[配置读写与 SESSION 存取]

  - \_\_construct()
    - 读入配置文件 json 串：$incFileContents = file(ROOT."comm/inc.php")
    - $incFileContents = $incFileContents[1];
    - 解析成 php 对象：$this->inc = json_decode($incFileContents);
  - readInc($name)
    - return $this->inc->$name;
    - //->readInc('appid') 既读取配置文件的 appid

- URL.class.php[基于 CURL 库的 get 与 post 请求]
  - combineURL($baseURL, $keysArr)
    - 拼接：$combined = $baseURL."?";
    - 拼接参数: foreach($keysArr as $key=>$val) $valueArr[] = "$key=$val";
    - 使用&拼接参数键值对：$keyStr = implode("&", $valueArr);
    - 生成形如http://xxx.com?a=b&c=d...的链接
  - get($url,$keysArr) 发送 get 请求
  - post($url,$keysArr,$flag = 0) 发送 post 请求
- Oauth.class.php[Oauth 相关 URL 动态拼接与 token 操作]
  - qq_login() 拼接 QQ 登录页面 URL
    - 读取 appid: $appid = $this->recoder->readInc("appid");
    - 读取回调地: $callback = $this->recoder->readInc("callback");
    - 读取授权列表：$this->recorder->readInc("scope");
    - 生成唯一随机串防 CSRF 攻击
    - 原样返回参数：md5(uniqid(rand(), TRUE));
    - state 写入 session 中：$this->recorder->write('state',$state);
      =》拼接之后得到https://graph.qq.com/oauth2.0/authorize?response_type=code&client_id=[APPID]&redirect_uri=[REDIRECT_URI]&scope=[SCOPE]&state=[STATE]
    - 跳转到生成的 url: header("Location:$login_url");
  - qq_callback() QQ 登录完成后的回调处理

### 7.SDK 优化

- SDK 太老，很久无人维护
  - 调整文件及目录结构
- SDK 中的常量名太常见，可能和现有项目冲突
  - 批量替换 SDK 中常量名称为不常见名称

### 8.整合 SDK 到 Web 项目中--请求访问 QQ 登录页面

```php
$oauth = new Oauth();
$oauth->qq_login();
```

### 9.整合 SDK 到 Web 项目中--获取 code 和 AccessToken

- 回调地址：http://test.open.mypro.com/callback.php

- 拿到返回的`code`，并请求 AccessToken

```php
$oauth = new Oauth();
$accessToken = $oauth->qq_callback();
```

### 9.整合 SDK 到 Web 项目中--获取 openID

(1) 关于 openId

- QQ 用户在第三方站点的唯一标识
- 同一个 QQ 用户在不同站点使用 QQ 登录 openId 始终一样

```php
$openid = $oauth->get_openid();
```

(2)存储`accesstoken`和`openid`到`cookie`中

```php
// 有效期时长可以读取session中的相应信息的有效期 [手动设置时需要将该时长小于实际有效期]
setcookie('qq_accesstoken', $accesstoken, time()+86400);
setcookie('qq_openid', $openid, time()+86400);
```

### 10.API 调用示例

#### 调用`get_user_info`接口，获取用户信息

(1)回调成功后，跳转到`index.php`文件

```php
header('Location: index.php');
```

(2)判断当前登录状态[通过 cookie]

- 未登录

  - 进行登录 [获取 AccessToken，获取 openid]

- 已经登录
  - 调用接口，获取信息

```php
$qc = new QC($_COOKIE['qq_accesstoken'], $_COOKIE['qq_openid']);
$userInfo = $qc->get_user_info();
```

### 平台政策与注意事项

- APPID 申请之后 3 个月未申请上线将被回收

- 申请上线需要使用官网提供的 QQ 登录按钮素材

- 遵守国家法律法规, 拒绝黄赌毒
