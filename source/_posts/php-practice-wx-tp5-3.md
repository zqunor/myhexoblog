---
title: 【实战】tp5+小程序(三)--微信登录与令牌
date: 2018-07-11 16:26:58
tags:
  - thinkphp5
  - 微信小程序
category:
  - 【PHP框架】
---

ThinkPHP5 从入门到深入学习，结合实战项目深入理解 ThinkPHP5 的特性和使用方法。深入学习 api 开发，学习微信登录和令牌的相关知识，并理解微信登录流程，完成与微信开放 api 之间的数据交互，完善项目的相应功能。
理解第三方登录授权的 code 和 token 交互过程。

<!--more-->

### 9-1 初识 Token - 意义与作用

说明：目前这种情况下，用户只要知道了系统的接口的形式，就可以直接访问，并获取数据，而大多数情况下，我们需要对用户身份进行验证，如：需要用户登录后才能访问的接口，以及需要管理员才能访问的接口等。

1.获取令牌

```info
客户端=》(账号、密码)=》getToken 《==》 账号、密码、Token、Auth
```

描述：客户端携带账号和密码信息，调用`getToken`接口，经过处理验证后，返回账号、密码、Token、Auth 等信息。

2.访问接口

```info
客户端=》(Token)=》下单接口 《==》 账号、密码、Token、Auth
```

验证：1.是否合法 2.是否有效 3.是否有操作权限

3.上面两个过程的 getToken 接口和下单接口就是被保护的接口，需要验证通过才能让用户访问。

### 9-2 微信登录流程

1.微信身份登录体系

![微信登录流程](![](https://ws1.sinaimg.cn/large/005EgYNMgy1fuchef1j72j30n10b9mza.jpg)

2.Token 在接口验证时的使用流程

![Token访问下单接口](https://ws1.sinaimg.cn/large/005EgYNMgy1fuchef3nc9j30gx058aao.jpg)

### 9-3 实现 Token 身份权限体系

1.获取 token 的请求使用 post 方法[安全性方面考虑]

2.将复杂的业务分层到`service`层[实现分层思想]

使用模型处理数据库 CRUD 相关的操作，对于不操作数据库的复杂业务，将其封装到 Service 目录下，实现分层处理的思想，Service 层是在 Model 层之上的业务层。

3.基础实现

1）控制器的定义

```php
// api/controller/v1/Token [注意命名空间]
public function getToken($code = '') {}
```

2）路由定义

```php
// route.php
Route::post('api/:version/token/user', 'api/:version.Token/getToken');
```

3）验证器校验

```php
// api/controller/v1/Token
(new TokenGet())->goCheck();
```

```php
// api/validate/TokenGet
protected $rule = [
    // 在验证器基类中定义isNotEmpty()方法
    'code' => 'require|isNotEmpty'
];

protected $message = [
    'code' => 'code必填！'
];
```

### 9-4/5/6/7 实现 Token 身份权限体系

1.获取微信生成的 code 码，并将其作为参数，传递给微信接口来获得 openid 和 access_token 等相关信息[openid/session_key]

```php
// api/controller/v1/Token
$userToken = new UserToken($code);
$token = $userToken->get();
```

**2.封装 Service 层，实现 Token 令牌的获取**[重点]

1） 配置微信小程序相关参数[app_id app_secret login_url]

2.1.1 在配置文件中设置微信小程序的相关参数

```php
// config/extra/wx.php
return [
    'app_id' => 'XXXXXXXXX',
    'app_secret' => 'XXXXXXXXX',
    'login_url' => "https://api.weixin.qq.com/sns/jscode2session?" . "appid=%s&secret=%s&js_code=%s&grant_type=authorization_code"
];
```

2.1.2 创建 Service 层的 UserToken 处理类，定义参数为私有属性

```php
// api/service/UserToken.php
namespace app\api\service;

use app\lib\exception\WechatException;
use app\lib\exception\TokenException;

class UserToken extends Token
{
    protected $code;
    protected $appid;
    protected $appSecret;
    protected $loginUrl;
}
```

2） 拼接参数，并使用 curl 模拟 http 请求微信服务器，并获取返回结果

```php
// api/service/UserToken.php
public function __construct($code)
{
    $this->code      = $code;
    $this->appid     = config('wx.app_id');
    $this->appSecret = config('wx.app_secret');
    $this->loginUrl  = sprintf(
        config('wx.login_url'),
        $this->appid, $this->appSecret, $this->code
    );
}

public function get()
{
    $result = curl_get($this->loginUrl);
}
```

在公共方法文件中定义 curl 模拟 http 请求的方法：

```php
// application/common.php
function curl_get($url, &$httpCode = 0)
{
    //1、初始化curl
    $curl = curl_init();

    //2、告诉curl,请求的地址
    curl_setopt($curl, CURLOPT_URL, $url);
    //3、将请求的数据返回，而不是直接输出
    curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);

    curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, 0);
    curl_setopt($curl, CURLOPT_CONNECTTIMEOUT, 10);

    $fileContents = curl_exec($curl); // 执行操作
    curl_close($curl); // 关键CURL会话

    return $fileContents; // 返回数据
}
```

3） 请求微信接口失败[微信内部错误/程序编写出错]的异常处理

```php
// api/service/UserToken.php get()
$wxResult = json_decode($result, true);

if (empty($wxResult)) {
    // 经验总结得：如果返回的结果为空[没有返回错误信息和错误代码]，则是微信服务器接口的问题，直接抛出异常一颗
    throw new \Exception('获取session_key及openID异常，微信内部错误');
} else {
    $loginFail = isset($wxResult['errcode']);
    // 程序传递的参数出错时，微信服务器会返回错误码和错误提示信息
    if ($loginFail) {
        $this->processLoginErr($wxResult);
    }
}
```

调用微信 Token 请求接口调用出错时的处理：

```php
// api/service/UserToken.php
private function processLoginErr($wxResult)
{
    throw new WechatException(
        [
            'msg'       => $wxResult['errmsg'],
            'errorCode' => $wxResult['errcode'],
        ]
    );
}
```

4） **成功获取微信接口返回数据后的操作[存储 openid、生成令牌、写入缓存、返回令牌]**

```php
// api/service/UserToken.php get()
return $this->grantToken($wxResult);
```

2.4.1 存储 openid

```php
// api/service/UserToken.php
private function grantToken($wxResult)
{
    $now = time();
    // 1.拿到openid
    $openid     = $wxResult['openid'];
    // $sessionKey = $wxResult['session_key'];

    // 2.查看数据库中该openid的记录是否已经存在[同一个用户的openid始终保持不变]
    $user = model('user')->getByOpenId($openid);

    // 3.如果存在，则不处理； 如果不存在，那么新增一条user记录
    if ($user) {
        $uid = $user->id;
    } else {
        $uid = $this->newUser($openid);
    }
}
```

根据 openid 查询是否已经存在该用户

```php
// api/model/User.php
public static function getByOpenId($openid)
{
    $user = self::where('openid', '=', $openid)->find();

    return $user;
}
```

创建用户

```php
// api/service/UserToken.php
private function newUser($openid)
{
    $user = model('user')->create([
       'openid' => $openid
    ]);

    return $user->id;
}
```

2.4.2 准备缓存数据(缓存的值)

[微信返回数据(openid|session_key) + uid(用户服务器中保存的用户记录 id) + scope(用户权限，值越大，权限越高) ]

```php
// api/service/UserToken.php  grantToken()
// 4.生成令牌，准备缓存数据，写入缓存 [获取用户的相关信息]
// 4.1 准备缓存数据
$cachedValue = $this->prepareCachedValue($wxResult, $uid);
```

准备缓存数据值的方法[缓存的值]

```php
// api/service/UserToken.php
private function prepareCachedValue($wxResult, $uid)
{
    $cachedValue = $wxResult;
    $cachedValue['uid'] = $uid;
    $cachedValue['scope'] = 16; // 数值越大，权限越多

    return $cachedValue;
}
```

2.4.3 写入缓存[令牌+微信返回数据+有效期]

```php
// api/service/UserToken.php  grantToken()
// 4.2 写入缓存，并返回令牌
$token = $this->saveToCache($cachedValue);
```

2.4.3.1 生成令牌(缓存的键) [随机字符串+时间戳+盐]

```php
// 令牌是用户程序生成的随机字符串，与微信服务器无关
// api/service/UserToken.php  saveToCache()
$key = self::generateToken();
```

在服务器层构建 Token 基类，处理用户登录 Token 和后续的其他 Token 信息[service 下 UserToken 继承该基类]

```php
// api/service/Token.php
public static function generateToken()
{
    // 用三组字符串，进行md5加密 [加强安全性]
    // 1.32个字符组成一组随机字符串
    $randChars = getRandChar(32);
    // 2.时间戳
    $timestamp = $_SERVER['REQUEST_TIME_FLOAT'];
    // 3.盐
    $salt = config('secure.token_salt');

    return md5($randChars.$timestamp.$salt);
}
```

公共方法中定义生成指定长度的随机字符串

```php
// application/common.php
function getRandChar($length)
{
    $str    = null;
    $strPol = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz";
    $max    = strlen($strPol) - 1;

    for ($i = 0; $i < $length; $i++) {
        $str .= $strPol[rand(0, $max)];
    }

    return $str;
}
```

创建安全配置文件[盐：随机字符串]

```php
// extra/secure.php
return [
    'token_salt' => 'E7epHZhrTfgQ'
];
```

2.4.3.2 配置文件中设置 cache 缓存的有效期

````php
创建安全配置文件[盐：随机字符串]

```php
// extra/setting.php
'token_expire_in' => 7200
````

2.4.3.3 创建缓存文件

```php
private function saveToCache($cachedValue)
{
    $key = self::generateToken();
    $value = json_encode($cachedValue);
    // 设置缓存失效时间
    $expire_in = config('setting.token_expire_in');

    $request = cache($key, $value, $expire_in);
    if (!$request) {
        // 令牌缓存出错
        throw new TokenException([
            'msg' => '服务器缓存异常',
            'errorCode' => 10005
        ]);
    }

    return $key;
}
```

2.4.4 返回令牌

```php
// api/service/UserToken.php  grantToken()
// 4.3 写入缓存，并返回令牌
return $token;
```

3.异常处理类

3.1 微信内部错误[直接抛出异常]

3.2 微信接口调用出错[微信相关异常处理类 WechatException]

```php
class WechatException extends BaseException
{
    public $code = 404;
    public $msg = '微信服务器接口调用失败';
    public $errorCode = 999;
}
```

3.3 缓存 Token 出错[Token 异常处理类 TokenException]

```php
class TokenException extends BaseException
{
    public $code = 401;
    public $msg = 'Token已过期或无效Token';
    public $errorCode = 10001;
}
```

### 9-8 获取请求参数 code 并调用 PHP 接口[借助微信开发工具]

#### 1.微信开发者工具中配置：

> 设置好 app_key 后，需要将 “详情” 中的 “不校验合法域名、web-view(业务域名)、TLS 版本以及 HTTPS 证书” 勾选上（在本地测试，没有远程访问的服务器或远程服务器访问的域名没有 https 证书）

#### 2.小程序代码：

(1) 在 config 中定义 restUrl

```javascript
// Protoss/utils/config.js [设置本地测试的域名基地址]
Config.restUrl = "http://mypro.com/api/v1/";
```

(2)在登录方法中获取 code

```javascript
// 在小程序登录调用wx.login()方法中输出code，然后使用接口请求工具将code作为post请求的参数，进行调用

// Protoss/utils/token.js getTokenFromServer()
wx.login({
  success: function(res) {
    console.log("code: " + res.code);
  }
});
```

#### 3.请求 PHP 接口获取 Token

```javascript
// 引用使用es6的module引入和定义
// 全局变量以g_开头
// 私有函数以_开头

import { Config } from "config.js";

class Token {
  constructor() {
    this.tokenUrl = Config.restUrl + "token/user";
  }

  verify() {
    var token = wx.getStorageSync("token");
    if (!token) {
      this.getTokenFromServer();
    }
  }

  getTokenFromServer(callBack) {
    var that = this;
    wx.login({
      success: function(res) {
        console.log("code: " + res.code);
        wx.request({
          url: that.tokenUrl,
          method: "POST",
          data: {
            code: res.code
          },
          success: function(res) {
            console.log("token： " + res.data.token);
            wx.setStorageSync("token", res.data.token);
            callBack && callBack(res.data.token);
          }
        });
      }
    });
  }
}

export { Token };
```

**【补充说明】**：

(1) 需要调试时，将 XDEBUG 参数拼接到`this.tokenUrl`即可

(2) 如果没有输出 code, 需要关闭开发者工具后再重新启动，会自动调用该方法，并输出 code
[调用过生成的 token 已经被存储到浏览器的 Storage 中，便不会再调用 Token 请求接口，从而不产生 code]

### 9-9 商品详情接口

(1) 定义控制器方法 `getOne($id)`

(2) 定义路由 `api/:version/product/:id`

(3) 模型类实现[隐藏部分字段、设置数据表关联、实现数据库查询]

> Product => properties => ProductProperty => 商品属性值[品名、口味、产地、保质期]
> Product => imgs => Image => 商品主图
> ProductImage => imgs.imgUrl => Image => 商品详情图

(4) 异常处理信息提示

```php
[
    'msg'       => '当前产品无详情',
    'errorCode' => 20001
]
```

### 9-10-1 路由变量规则

1.路由匹配规则在项目中的应用。

```php
Route::get('api/:version/product/recent', 'api/:version.Product/getRecent');
Route::get('api/:version/product/:id', 'api/:version.Product/getOne');
```

2.存在的问题

目前调用接口都不存在问题，但是当将`:id`行放到`recent`行之前后，在调用`recent`路由时，则会因为优先匹配`:id`对应的路由，
此时则会因为参数校验不通过而报错。

3.解决之道：

对路由匹配规则进行限定，设置变量规则，对于`:id`行，限定只有当参数为数值时才匹配到当前行。即设置 `$id`的变量规则

变量规则：为变量用正则的方式指定变量规则，弥补了动态变量无法限制具体的类型问题，并且支持全局规则设置。

4.代码实现[设置变量规则]

```php
Route::get('api/:version/product/:id', 'api/:version.Product/getOne', [], ['id'=>'\d+']);
```

### 9-10-2 路由分组

对路由配置文件中，具有相同路由前缀的路由归为同一路由组，例如：

对于几个对应产品信息的路由，

```php
Route::get('api/:version/product/recent', 'api/:version.Product/getRecent');
Route::get('api/:version/product/by_category', 'api/:version.Product/getAllInCategory');
Route::get('api/:version/product/:id', 'api/:version.Product/getOne');
```

可以分组到产品组路由下，

```php
// 闭包方式注册路由分组
Route::group('api/:version/product', function() {
    Route::get('recent', 'api/:version.Product/getRecent');
    Route::get('by_category', 'api/:version.Product/getAllInCategory');
    Route::get(':id', 'api/:version.Product/getOne', [], ['id' => '\d+']);
});
```

或者：

```php
// 数组方式注册路由分组
Route::group('api/:version/product', [
    'recent' => ['api/:version.Product/getRecent'],
    'by_category' => ['api/:version.Product/getAllInCategory'],
    ':id' => ['api/:version.Product/getOne', [], ['id' => '\d+']]
],['method' => 'get']);
```

路由分组的方式定义路由，执行的效率会比一般形式高一点。

【注】路由分组的公共路由定义时，不能在末尾加`/`，否则会报控制器不存在的错误

### 9-11 闭包函数构建查询器

1.完成的商品详情的数据信息格式为：

```info
{
"id": 11,
"name": "贵妃笑 100克",
"price": "0.01",
"stock": 994,
"main_img_url": "http://mypro.com/static/images/product-dryfruit-a@6.png",
"summary": null,
"img_id": 39,
"imgs":[
    {
        "id": 4,
        "order": 1,
        "img_url":{
            "url": "http://mypro.com/static/images/detail-1@1-dryfruit.png"
        }
    },
    {
        "id": 5,
        "order": 2,
        "img_url":{
            "url": "http://mypro.com/static/images/detail-2@1-dryfruit.png"
        }
    },
],
"properties":[
    {
        "id": 1,
        "name": "品名",
        "detail": "杨梅"
    },
]
}
```

2.问题：其中`imgs`的值为每个商品下的所有图片介绍，所以所有图片之间一定存在一定的顺序，其中`imgs`数组下的数据中存在`order`排序字段，如何对`imgs`的数据通过`order`进行排序？

3.【答】：使用闭包函数构建查询器【相当于拼接 sql】。

```php
$product = self::with([
        'imgs' => function($query) {
            $query->with(['imgUrl'])->order('order asc');
        }
    ])
    ->with(['properties'])
    ->find($id);
```

4.思路分析：

（1）要对 imgs 下的数据进行处理，需要获取到每组数据，然后对`order`字段进行排序。【通过闭包函数获取到每组数据】

（2）除了要对每组数据进行按`order`排序，还需要处理`img_url`。【通过 with 链式操作处理`img_url`】

5.关于闭包函数的理解：

```php
'imgs' => function($query) {
    $query->with(['imgUrl'])->order('order asc');
}
```

对于数组`imgs`，通过闭包函数，获取到每组数据，其中`$query`即作为参数接收每组数据的值，然后再对每组数据的`img_url`通过 with 进行数据关联。

### 9-12 用户收货地址

1.需求说明：

用户收货地址接口信息需要进行身份验证，登录用户只能查看和操作自己的地址信息，未登录用户不能访问。

为简化操作，当前将用户和用户地址的关联关系设定为一对一。

2.思考点：

（1）对登录状态的判断：

当用户访问小程序时，调用`wx.login()`方法，并生成`code`,后台接口拿到 code 后生成 token，并用 token 以及配置的`app_id`和`app_secret`请求微信接口，并获取微信返回的`openid`等信息，存储到缓存中
[以 token 为键，uid|wxResult|scope 组成的 json 数据为值]

所以，创建或修改用户地址信息时，在处理地址信息和用户信息的关联时，使用的用户信息，应当是当前登录用户的信息，而不能是客户端传递的用户信息参数[可能传递有误，导致误操作到其他用户的地址信息]

实现一定程度上的接口保护。

（2）传入参数的检验

验证器校验往往只能验证某个字段或某些字段的合法性，而客户端可能传入的参数比需要的参数多，或者传入了`uid`或者`user_id`，导致更新时覆盖了其他用户的数据信息，对系统的安全性造成影响，
所以，在接收客户端传入参数时，需要进行多余字段的过滤。

（3）对手机号的验证

正则表达式的应用场景，正则模式`^1(3|4|5|6|7|8)[0-9]\d{8}$^`

（4）**通过模型关联，实现用户地址的新增和更新**【新】

通过关联模型方法，创建数据

```php
// 新增
$user->address()->save($dataArray);
```

通过关联模型属性，对当前属性对应的记录进行更新

```php
// 更新
$user->address->save($dataArray);
```

（5）模型关联方法的选择：

模型关联方法的区分：

    有主键关联无主键 =》 belongsTo
    无主键关联有主键 =》 hasOne|hasMany

（6）HTTP 状态码

200：操作成功，服务器已成功处理了请求。说明：[如果是对您的 robots.txt 文件显示此状态码，则表示 Googlebot 已成功检索到该文件](https://blog.csdn.net/u014028956/article/details/47125403)

201：创建成功，表示服务器执行成功，并且创建了新的资源

设置接口调用成功后的状态码标识：

```php
return json(new SuccessMessage(), 201);
```

### 9-12-1 通过令牌获取用户标识

(1)定义控制器方法 `createOrUpdate()`

(2)定义路由 `api/:version/address`

(3)验证器验证用户输入数据 [`name`, `mobile`, `province`, `city`, `country`, `detail`]

(4)异常处理信息提示

当数据不合法时抛出异常，而当操作成功时，也需要返回相应的数据信息。当前项目将抛出的成功信息也放在异常处理类库下。

### 9-12-2 面向对象的方式封装获取 uid 方法

1.通过令牌 token 即可获取缓存中对应的用户信息，而缓存中的信息包括`uid` `scope` `wxResult`[`openid` `session_key`]

而在 http 请求时，token 保存在 header 头信息中，获取头信息中`token`的方法：

`$token = Request::instance()->header('token');`

2.通过 json 键值对的键，获取 cache 数据

`Cache::get($token)`

3.增强项目的扩展性，可将通过 token 获取变量的方法进行封装。

4.代码实现：

```php
public static function getCurrentTokenVar($key)
{
    $token = Request::instance()->header('token');
    $vars  = Cache::get($token);
    if ( ! $vars) {
        throw new TokenException();
    } else {
        if (!is_array($vars)) {
            $vars = json_decode($vars, true);
        }

        if (isset($vars[$key])) {
            return $vars[$key];
        } else {
            throw new Exception('尝试获取的Token变量不存在');
        }
    }
}

public static function getCurrentUid()
{
    $uid = self::getCurrentTokenVar('uid');
    return $uid;
}
```

### 9-12-3 模型新增和更新

通过用户模型，进行面向对象方式的新增和更新

（1）user 模型定义 address()关联方法，获取到用户地址信息，当用户地址信息不存在时，也通过**关联模型方法**，保存地址信息

```php
// 新增
$user->address()->save($dataArray);
```

（2）user 模型通过 address()关联方法关联 user_address 数据表中对应的用户地址信息，通过关联获取的数据仍然可以作为模型的属性值使用，
再通过**关联模型属性**，对当前属性对应的记录进行更新 [包含主键 id]

```php
// 更新
$user->address->save($dataArray);
```

### 9-12-4 参数过滤

封装处理客户端传入的参数的方法，由于当前用户的信息是通过缓存获取的，为避免用户传入的参数造成错误修改，所以需要对客户端传入数据进行过滤，
如果携带用户 id 参数，则抛出异常，不再继续处理。除此之外，对于传入的无效、多余数据，进行过滤，仅接收验证器需要验证的字段信息。

```php
public function getDataByRule($params)
{
    if (isset($params['uid']) || isset($params['user_id'])) {
        throw new ParameterException([
            'msg' => '参数中包含非法的参数名user_id或者uid'
        ]);
    }
    $newArray = [];
    foreach ($this->rule as $key => $value) {
        $newArray[$key] = $params[$key];
    }

    return $newArray;
}
```

### 9-12-5 接口测试

1.需要的参数

- token: header 请求头 [通过微信小程序的开发者工具]

- address 字段信息 [`name`, `mobile`, `province`, `city`, `country`, `detail`]

2.返回的数据

```json
{
  "code": 201,
  "msg": "ok",
  "errorCode": 0
}
```

并且通过设置返回值为带状态码的 json 数据，`json(new SuccessMessage(), 201)`，可将 http 的状态码也设置为`201`
