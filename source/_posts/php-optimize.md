---
title: PHP性能优化
date: 2018-07-09 08:57:55
tags:
  - optimize
category:
  - 【PHP相关】
---

PHP 运行环境的性能考虑在 php 深入学习中需要逐步强化意识，并着手实现，其中对于性能分析的相关工具也需要有一定的掌握，比如压力测试工具 Apache Benchmark，Opcode 代码分析工具 vld，PHP 性能分析工具 XHProf，另外，对于日常编写代码时，也需要考虑 PHP 自身的特性，进行扬长避短，使用 isset 而不用 array_key_exists 方法，以及尽可能规避 PHP 自带的魔术方法。对 PHP 的运行流程也需要有一个大致的了解，知道 Opcode 在 PHP 执行过程中的阶段。需要逐渐加深对 PHP 深层次的思考。

<!--more-->

# PHP 性能优化

## 一、语言级性能优化(一)

PHP 性能问题的解决方向

    PHP语言级别的性能优化 =》 PHP周边问题的性能优化 =》 PHP语言自身分析、优化

### 1.压力测试工具 Apache Benchmark (ab)

#### (1)测试工具基本介绍

1). ab 是由 Apache 提供的压力测试软件，安装 apache 服务器时会自带该压力测试软件

2). 基本使用[Linux 平台]

```bash
./ab -n1000 -c100 http://www.baidu.com/

# -n 请求数
# -c 并发数
# url 目标压力测试地址
```

3). 参考项

```bash
# 每秒接收请求数 =》 尽可能大
Requests per second (mean)

# 每一个请求的耗时情况 =》 尽可能小
Time per request (mean, across all concurrent requests)
```

### 2.PHP 自身能力

#### (1)优化点：少写代码，多用 PHP 自身能力

1). 性能问题：自身代码冗余较多，可读性不佳，并且性能低。

2). 为什么性能低：PHP 代码需要编译解释为底层语言，这一过程每次请求都会处理一遍，开销大。

3). 好的方法：多实用 PHP 内置变量、常量、函数。

#### (2)PHP 代码运行流程

```info
.php文件 =>[zend引擎 Scanner]=> zend exprs => [Parser] => Opcodes(要被执行的代码) => [Exec] => Output
```

【补充】目前很多 php 的缓存服务使用的都是 opcode，节省了扫描和解析的过程，提升速度。

#### (3)PHP 内置函数之间的性能测试

1). `array_key_exists()` vs `isset()`

php 执行效率上： isset > array_key_exists

> 【插曲】：在接触的项目中，大多数情况下确实使用的也是 isset()，但是记得某次看到同事写的代码中有 array_key_exists()方法时，自己查看了手册，确认了这个方法的使用方法后，还特地将 isset 换成 array_key_exists 方法进行编写代码，粗浅的认为 array_key_exists()方法要比 isset()方法高级，学习了性能分析之后，顿时觉得之前的认识很是浅陋，也意识到项目中之所有 isset()更为常见是合理的。

## 二、语言级性能优化(二)

### 1.优化点：减少 PHP 魔法函数的使用

#### (1). 情况描述：

PHP 提供的魔法函数，性能不佳

#### (2). 为什么性能低：

为了给 PHP 程序员省事，PHP 语言为你做了很多

#### (3). 好的方法：

尽可能规避使用 PHP 魔法函数

【补充】：命令行模式查看 php 文件执行耗时

```bash
# time命令
time php test.php

# 输出结果
real
user => 主要参考的耗时
sys
```

### 2.优化点： 产生额外开销的错误抑制符 `@`

#### (1).情况描述：

PHP 提供的错误已支付只是为了方便懒人

#### (2). `@` 的实际逻辑：

在代码开始前、结束后，增加 Opcode，忽略报错
vld - PHP Opcode 查看扩展

#### (3).错误抑制符的性能测试

1)测试文件`at.php`

```php
//at.php
file_get_contens('xxx'); // xxx文件不存在
```

2)使用 vld 扩展执行`at.php`，查看执行过程的完整 Opcode

`php -dvld.active=1 -dvld.execute=0 at.php`

```bash
# 不加@错误抑制符时Opcode的执行情况
SEND_VAL
DO_FCALL
RETURN

# 加@错误抑制符时Opcode的执行情况
BEGIN_SILENCE
SEND_VAL
DO_FCALL
END_SILENCE
RETURN
```

#### (4).好的建议：尽量不要使用@错误抑制符

可以使用 try throw 方式进行错误控制

### 3.优化点： 合理使用内存

#### (1)情况描述：

PHP 有内存回收机制保底，但也要小心使用内存

#### (2)好的建议：

利用 unset()及时释放不使用的内存[注：unset()出现注销不掉的情况]

### 4.优化点： 尽量少的使用正则表达式

#### (1) 情况描述：

正则表达式的回溯开销比较大，"没有金刚钻别揽瓷器活"

#### (2) 好的建议：

利用字符串处理函数，实现相同逻辑

### 5.优化点：避免在循环内做运算

#### (1) 情况描述：

循环内的计算式将会被重复计算

#### (2) 代码示例：

```php
$str = "hello world";
for ($i=0; $i < strlen($str); $i++) {
    # code...
}

// 其中strlen()方法会在每次循环时计算一次

// 进行优化
$str = "hello world";
$strlen = strlen($str);
for ($i=0; $i < $strlen; $i++) {
    # code...
}
```

### 6.优化点： 减少计算密集型业务

#### (1) 情况描述：

PHP 不适合密集型运算的场景[大批量日志分析、大批量数据处理]

#### (2) 为什么？

PHP 语言特性决定了 PHP 不适合做大数据量运算。[需要解析成 C 语言进行运算，C 语言可能几行代码就实现的计算，php 可能需要很多行代码才能实现]

#### (3) PHP 适用场景：

适合衔接 Webserver 与后端服务、UI 呈现[纽带]

### 7.优化点： 务必适用带引号字符串做键值

#### (1) 情况描述：

PHP 会将没有引号的键值当做常量，产生查找常量的开销

【补充】：

```bash
# 将后台运行的任务放到前台终端运行
fg

# 相关命令： jobs  bg  fg

# 将任务号为1的任务从后台执行转换到其那台执行。
fg 1

# 执行上述命令后，命令行窗口将显示如下信息
find / -name password
```

#### (2) 程序说明

```php
define('key', 'imooc');

$array = array(
    'key' => 'hello world!',
    'imooc' => 'http://www.imooc.com/'
);
echo $array["key"] . '\n'; // 输出 hello world
echo $array[key] . '\n'; // 输出 http://www.imooc.com/
```

【说明】：

当时用`$array[key]`时，程序也把 key 作为常量去查找，当查找到时，获取到常量的值；当没查找到时，再到数组内部，将其作为键 key 字符串进行解析

#### (3) 好的建议：

严格使用引号作为键值

## 三、PHP 周边问题的分析与阐述

### 1.PHP 周边范围：

Linux 运行环境

文件存储[磁盘]

数据库[mysql]

缓存[硬件的内存、php 缓存技术：memcache redis]

网络

### 2.PHP 周边对 PHP 程序的影响分析

#### (1) 连接数据库操作

1） 同一台服务器 => 数据库优化 决定时间性能

2） 分布式服务器 => 数据库优化 + 网络速度 决定时间性能

#### (2) 减少文件类操作

1） 常见 PHP 场景的开销次序：

读写磁盘、 读写数据库、读写内存、读写网络数据

2） 时间开销：

        读写内存 <<(远小于) 读写数据库[基于文件系统，操作本地磁盘] <(小于) 读写磁盘 < 读写网络数据

        数据库会使用内存作为缓存，将其热数据先缓存在内存中，异步地写入到数据库 =》 数据库介于内存和磁盘之间

        网络数据：通过socket发起，socket使用的是本地的文件句柄，磁盘操作。受网络延迟影响，延迟大时远远小于读写磁盘，延迟小时和读写磁盘差不多。

3） 总结：

尽可能多的使用读写数据库、读写内存，尽量规避操作磁盘和操作网络数据。

#### (3) 优化网络请求

网络请求的坑：

1.对方接口的不确定因素 2.网络稳定性

如何优化网络请求？

1.设置超时时间
a) 连接超时
b) 读超时
c) 写超时

2.将串行请求并行化
a) 使用 curl*multi*\*() => 最简单，但是并不是最好用
b) 使用 swoole 扩展 => 效果更好

#### (4) 压缩 PHP 接口输出

1） 如何压缩？

使用 Gzip 即可。

2） 利与弊：
利： 利于我们的数据输出，Client 端能更快获取数据
弊： 额外的 CPU 开销

#### (5) PHP 缓存复用

1）什么情况下做输出内容的缓存？

多次请求，内容不变的情况。[模板缓存]

          |---Cached----/\
         \/              |
        .php <-------> Cache -> No Cache
                        /\          |
                         |---------\/

#### (6) 重叠时间窗口思想

串行：

    process1[客户端]  process2[web server]  process3[php]  process4[mysql或缓存]

重叠时间窗口：

    process1
        process2
            process3
                process4

使用前提：后一个任务不强依赖于前一个任务的输出或返回。

#### (7) PHP 旁路处理方案

    x.php                         x.php
      |                             |
    process1                      process1
      |                             |---------process2
    process2                      process3       |
      |                             |<------------
    process3                      process4
      |
    process4

使用前提：后一个任务不强依赖于前一个任务的输出或返回。

## 四、具体的性能分析

### 1. 借助 xhprof 工具分析 PHP 性能

工具： XHProf [源自 Facebook 的 PHP 性能分析工具]

实践： 分析 Wordpress 程序，做优化

#### (1)准备工作

1） 检查 xhprof 工具是否安装成功

```bash
php --ri xhprof

//输出
xhprof
xhprof => 0.9.2
CPU num => 1
```

2） 在 wordpress 代码中[index.php 文件]

    1.在起始位置添加`xhprof_enable()`进行开启xhprof

    2.在执行结束位置添加`xhprof_disable()`返回性能分析数据

    3.添加xhprof的两个lib文件

wordpress/index.php 代码

```php
xhprof_enable();

define('WP_USE_THEMES', true);

require(dirname(__FILE__) . '/wp-blog-header.php');

$data = xhprof_disable();

include_once "/var/www/html/xhprof_lib/utils/xhprof_lib.php";
include_once "/var/www/html/xhprof_lib/utils/xhprof_runs.php";

$objXhprofRun = new XHProfRuns_Default();
$run_id = $objXhprofRun->save_run($data, "test");
var_dump($run_id);
```

3） wordpress 同级目录下的 xhp 目录

    callgraph.php
    css/
    docs/
    index.php
    jquery/
    js/
    typeahead.php

4）浏览器访问 xhp/index.php

    查看列表形式的性能分析 =》 View Full Callgraph =》 查看图形流程分析[查看耗时最长的文件和方法]=>MO::import_from_reader| MO::make_entry

5）查找最需要优化的文件

`grep 'import_from_reader' ./ -r`

6）主要耗时点：`MO::make_entry()` -- 执行的是多语言支持的功能

优化方式：直接注释掉调用的相关代码

【补充】：`MO`和`PO`都是 PHP 处理多语言程序的语言包

7）优化结果：

让程序不再是只有一个或一处相互关联的高消耗代码点。

### 2. PHP 性能分析工具扩展

XHProf -- PHP 性能分析工具

ab -- 压力测试

vld -- opcode 代码分析

## 五、PHP 性能瓶颈终极办法

### 1.Opcode Cache: PHP 扩展`APC`

官方地址：[前往](http://pecl.php.net/package-search.php?pkg_name=apc&bool=AND&submit=Search)

已不再更新。

【补充】：`pecl.php.net`是 php 扩展的官方维护站点。了解并使用[类似 APC 的 opcode 缓存扩展](http://pecl.php.net/packages.php?catpid=3&catname=Caching)？

yac - 通过共享内存来缓存用户数据，用于代替 APC 和本地 memcached 的方案。

### 2.扩展实现：通过 PHP 扩展代替原 PHP 代码中高频逻辑

### 3.Runtime 优化： HHVM
