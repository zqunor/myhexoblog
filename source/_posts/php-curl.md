---
title: PHP扩展功能--cURL
date: 2018-04-16 13:32:53
tags:
    - php
    - curl
category:
    - Server
    - PHP
---

cURL表示以命令行的形式请求某个url, 提交数据或获取相应数据。在日常的程序开发中会用到，因此，了解cURL的原理和过程，有助于实际工作和项目中的应用。

<!--more-->

## 一、入门三部曲

### 1、cURL 是什么？

[wikipedia 介绍][1]：

    * cURL是一个利用URL语法在命令行下工作的文件传输工具，1997年首次发行。它支持文件上传和下载，所以是综合传输工具，但按传统，习惯称cURL为下载工具。cURL还包含了用于程序开发的libcurl。
    * cURL支持的通信协议有FTP、FTPS、HTTP、HTTPS、TFTP、SFTP、Gopher、SCP、Telnet、DICT、FILE、LDAP、LDAPS、IMAP、POP3、SMTP和RTSP。
    * libcurl支持的平台有Solaris、NetBSD、FreeBSD、OpenBSD、Darwin、HP-UX、IRIX、AIX、Tru64、Linux、UnixWare、HURD、Windows、Symbian、Amiga、OS/2、BeOS、Mac OS X、Ultrix、QNX、BlackBerry Tablet OS、OpenVMS、RISC OS、Novell NetWare、DOS等。

简而言之：cURL 是下载工具、传输工具。利用 url 的语法规则传输文件、数据的命令行工具和库。

<!--more-->

### 2、为什么要用 cURL?

通常是通过表单（html）提交数据到 php 文件从而实现数据的交互，但是不能实现**php 文件之间**的数据和文件传输，所以，cURL 的应用场景主要是 php 文件之间的数据和文件传输。

### 3、在 PHP 中怎么用 cURL?

（1）php.ini 中开启 curl 扩展

```ini
extension=php_curl.dll
```

然后重启 apache

（2）在 phpinfo()的输出信息中查看是否有 curl 的相关信息
[![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/phpinfo_curl.png)][2]

【注】：如果开启无效，可以尝试将 php 安装目录下的 libeay32.dll 、ssleay32.dll 拷贝到 windows 或 windows/system32 目录下

## 二、cURL 在 PHP 中的应用

### 必备函数：

(1)curl_init() --- 初始化 cURL 会话

(2)curl_setopt() --- 设置 cURL 传输选项

参数：

* post 方式：
  * CURLOPT_POST
  * CURLOPT_POSTFIELDS
* get 方式：
  * CURLOPT_RETURNTRANSFER
  * CURLOPT_SSL_VERIFYHOST
* 安全验证：

  * CURLOPT_SSL_VERIFYPEER
  * CURLOPT_SSL_VERIFYPEER

(3)curl_exec(); --- 执行 cURL 会话

(4)curl_close() --- 关闭 cURL 会话

### 1、模拟 get 请求

（1）默认是直接显示返回的数据，对于 html 数据，则直接以网页的形式显示。

```php
//1、初始化curl
$curl = curl_init();

//2、告诉curl,请求的地址
curl_setopt($curl, CURLOPT_URL, 'http://www.baidu.com/index.php');

//3、发送请求
curl_exec($curl);

//4、关闭资源
curl_close($curl);
```

（2）设置只获取数据，不直接显示

```php
//1、初始化curl
$curl = curl_init();

//2、告诉curl,请求的地址
curl_setopt($curl, CURLOPT_URL, 'http://www.baidu.com/index.php');
//将请求的数据返回，而不是直接输出
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);

//3、发送请求
$res = curl_exec($curl);
var_dump($res);

//4、关闭资源
curl_close($curl);
```

### 2、模拟 post 请求

```php
1、初始化curl
$curl = curl_init();

//2、设置请求的地址
curl_setopt($curl, CURLOPT_URL, 'http://localhost/curl_post.php');
// （1）设置请求的方式为post
curl_setopt($curl, CURLOPT_POST, true);
// （2）设置post提交的数据
$data = [
    'username' => 'zqunor',
    'password' => 'zqunor123'
];
// （3）提交数据
curl_setopt($curl, CURLOPT_POSTFIELDS, $data);

//3、发送请求
curl_exec($curl);

//4、关闭资源
curl_close($curl);
```

### 3、封装成类，兼容 post 和 get 方式

```php
class HttpRequest
{
    private static $isShow = false;

    public function __set($attr, $value)
    {
        $this->$attr = $value;
    }

    public static function send($url,$data=null)
    {
        $curl = curl_init();
        // 设置请求的url地址
        curl_setopt($curl, CURLOPT_URL, $url);

        // 直接跳过安全证书的验证
        curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, false);
        curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, false);

        // 根据$data判断是post还是get方式
        if (!empty($data)) {
            // 如果$data非空，则为post方式
            curl_setopt($curl, CURLOPT_POST, true);
            curl_setopt($curl, CURLOPT_POSTFIELDS, $data);
        }
        // 反之为get方式
        if (!self::$isShow) {
            // 不直接显示数据，而是以返回值的形式
            curl_setopt($curl, CURLOPT_RETURNTRANSFER, $url);
        }
        $res = curl_exec($curl);
        return $res;

        curl_close($curl);
    }
}
```

### 4、实例化进行数据获取

```php
// 调用封装的类，请求知乎php话题下的数据
$res = HttpRequest::send('https://www.zhihu.com/search?type=content&q=php');

// 查看需要获取的数据的html样式
// <a target="_blank" href="/question/26498147/answer/33029411" data-reactid="218"><span class="Highlight" data-reactid="219">「<em>PHP</em> 是最好的语言」这个梗是怎么来的？</span></a>
// <a target="_blank" href="/question/41913568/answer/95778872" data-reactid="366"><span class="Highlight" data-reactid="367">如何看待天猫彻底抛弃<em>PHP</em>？</span></a>
// <a target="_blank" href="/question/25038841/answer/44396770" data-reactid="292"><span class="Highlight" data-reactid="293"><em>PHP</em>、Java、Python、C、C++ 这几种编程语言都各有什么特点或优点？</span></a>

// 根据样式设置正则匹配模式，筛选所需数据
$reg = '/<a[^>]*><span class="Highlight"[^>]*>(.+?)<\/span><\/a>/';

preg_match_all($reg, $res, $match);

var_dump($match);
```

### 5、查看匹配结果

[![avatar](https://raw.githubusercontent.com/zqunor/MarkdownPic/master/curl_result.png)][3]

完成！

[1]: https://zh.wikipedia.org/wiki/CURL
[2]: https://raw.githubusercontent.com/zqunor/MarkdownPic/master/phpinfo_curl.png
[3]: https://raw.githubusercontent.com/zqunor/MarkdownPic/master/curl_result.png
