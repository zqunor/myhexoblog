---
title: windows7使用Sphinx+PHP+MySQL详细介绍
date: 2018-05-14 09:41:08
tags:
    - sphinx
    - windows
    - thinkphp5
category:
    - Server
    - PHP
    - Sphinx
---

由于业务需要，需要做类似淘宝商城商品检索的功能，对于数据量很大的情况，MySQL 查询的效率损耗很大，需要使用专门的索引引擎进行搜索查询，实现功能，对于和 PHP 和 Mysql 的结合的索引引擎中， xunsearch 和 sphinx 是较为著名的，但由于 xunsearch 服务器端不支持 windows，所以暂且先考虑 sphinx 的使用。sphinx 目前已支持简体中文、繁体中文和英文的检索，不需要额外安装插件支持。

<!--more-->

## 一、安装（Windows）

### **1.官方下载**

Sphinx 下载地址： [下载]

### **2.解压并重命名**

此处下载版本为`3.0.3`，将 sphinx 文件夹命名为`sphinx`

### **3.文件夹目录介绍**

```ini
sphinx
--api(各语言支持的api)
--bin（二进制程序）
--doc（文档说明）
--etc（配置文件：conf/sql）
--misc
--src
# 手动创建以下两个文件夹
--data
--log
```

### **4.设置配置文件**

(1)将`sphinx/etc/sphinx-min.conf.dist`文件复制到`sphinx/bin/`目录下，并重命名为`sphinx.conf`

注：`sphinx/etc/sphinx.conf.dist`为带注释的详细的

(2)设置配置项

主要是以下为配置函数：

> source src1{} --- 设置索引源(数据库的基本配置和数据表)

```ini
# 连接的数据库类型
type = mysql
# 连接的数据库主机
sql_host = localhost
# 数据库连接的用户名，默认为test
sql_user = root
# 数据库连接的密码，默认为空
sql_pass =123123
# 连接的数据库名称，默认为test
sql_db = test
# 连接数据库的端口号，默认为3306
sql_port = 3306
# 操作的数据表执行的查询语句
sql_query = \
    SELECT id, group_id, UNIX_TIMESTAMP(date_added) AS date_added, title, content \
    FROM documents

# 设置查询过滤条件
sql_attr_uint = group_id
```

> index test1{} --- 索引文件

```ini
# 指定索引源
source = src1
# 索引数据存放目录，默认为/var/data/test1
path = D:\Service\sphinx\data\test1

# 设置中文匹配
min_word_len    = 1
# 指定字符集（已废弃）
charset_type    = utf-8
# 指定utf-8的编码表
charset_table   = 0..9, A..Z->a..z, _, a..z, U+410..U+42F->U+430..U+44F, U+430..U+44F

min_prefix_len = 0
min_infix_len = 1

# 开启中文分词支持
ngram_len = 1
# 需要分词的字符
ngram_chars = U+3000..U+2FA1F
```

> index test1stemmed : test1{}

```ini
# 主要需要修改的配置项，默认为/var/data/test1stemmed

path = D:\Service\sphinx\data\test1stemmed
```

> index rt{}

```ini
# 主要需要修改的配置项，默认为/var/data/rt
path = D:\Service\sphinx\data\rt

# 指定对哪些字段进行匹配
rt_field = name
rt_field = ename
rt_field = setmeal
rt_field = category
rt_field = country
rt_field = traffic
rt_field = body

rt_attr_uint = offerid
```

> searchd{}

```ini
# 自定义日志文件位置
log = D:\Service\sphinx\log\searchd.log
query_log = D:\Service\sphinx\log\query.log
pid_file = D:\Service\sphinx\log\searchd.pid
```

以下几项不需要修改默认值，即可直接使用

> source src1throttled : src1{}

分布式索引的相关配置，没有则可以不修改

> index dist1{}
> indexer{}
> common{}

【注】：主要的配置是`source` `index` `indexer` `searchd`, 其他几项可以不需要

### **5.操作数据库，导入样例数据**

(1)进入到 mysql 命令行，执行命令

```bash
D:\phpStudy\PHPTutorial\MySQL\bin>mysql -uroot -p
Enter password: *************
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 12
Server version: 5.5.53 MySQL Community Server (GPL)


mysql> use test;

# 恢复样例数据到数据库
mysql> source /D:\Service\sphinx\etc/eaxmple.sql

# 新增两个数据表，documents和tags
mysql> show tables;
documents
tags
```

### **6.生成索引文件**

cmd 命令行进入到`sphinx/bin/`目录下

```bash
# 生成索引文件,本地重新构建--rotate
> indexer.exe --config sphinx.conf --all --rotate
Sphinx 3.0.3-dev (commit facc3fb)
Copyright (c) 2001-2018, Andrew Aksyonoff
Copyright (c) 2008-2016, Sphinx Technologies Inc (http://sphinxsearch.com)

using config file 'sphinx.conf'...
WARNING: key 'docinfo' was permanently removed from Sphinx configuration. Refer to documentation for details.
WARNING: key 'dict' was permanently removed from Sphinx configuration. Refer to documentation for details.
WARNING: key 'mva_updates_pool' was permanently removed from Sphinx configuration. Refer to documentation for details.
indexing index 'test1'...
collected 4 docs, 0.0 MB
sorted 0.0 Mhits, 100.0% done
total 4 docs, 0.2 Kb
total 1.0 sec, 0.2 Kb/sec, 3 docs/sec
indexing index 'test1stemmed'...
collected 4 docs, 0.0 MB
sorted 0.0 Mhits, 100.0% done
total 4 docs, 0.2 Kb
total 1.0 sec, 0.2 Kb/sec, 3 docs/sec
skipping non-plain index 'dist1'...
skipping non-plain index 'rt'...
```

【注】新版 sphinx 的 bin 目录下已经没有 search.exe 程序，所以不能直接在命令行执行返回结果，只能使用 api 接口返回数据。

### **7.开启搜索服务，保持后台运行**

```bash
> searchd.exe --pidfile

[Tue May 15 09:02:14.690 2018] [7776] using config file './sphinx.conf'...
listening on all interfaces, port=9312
listening on all interfaces, port=9306
Sphinx 3.0.3-dev (commit facc3fb)
Copyright (c) 2001-2018, Andrew Aksyonoff
Copyright (c) 2008-2016, Sphinx Technologies Inc (http://sphinxsearch.com)
```

## 二、PHP 开启 sphinx 扩展

### **1.下载 php_sphinx 扩展**: [前往]

具体需要下载的版本需要查看 phpinfo 信息:

    Architecture         ==》x86/x64
    PHP Extension Build  ==》NTS/NS

下载并解压后，将`php_sphinx.dll`文件放到 php/ext 目录下

### **2.修改 php.ini 配置文件**

```ini
# 在 Dynamic Extensions 列表中添加php_sphinx扩展
extension=php_sphinx.dll
```

修改后重启 apache 服务

### **3.在 phpinfo.php 输出的信息中查看 sphinx 扩展是否安装成功**

```info
            sphinx
sphinx support	 enabled
Version	         1.3.2
Revision	 $Revision$
```

## 三、代码实现

### 1.样例数据表`test.documents`记录：

```info
 id   group_id   group_id2   date_added             title             content
 1    1             5        2018-05-14 09:12:25   test one           this is my test document number one. also checking search within phrases.
 2    1             6        2018-05-14 09:12:25   test two           this is my test document number two
 3    2             7        2018-05-14 09:12:25   another doc        this is another group
 4    2             8        2018-05-14 09:12:25   doc number four    this is to test groups
```

### 2.PHP 代码实现

#### **一般实现**

```php
<?php
require('sphinxapi.php');

$sphinx = new SphinxClient();
$sphinx->SetServer('localhost',9312);
$sphinx->SetMatchMode(SPH_MATCH_ANY);
$sphinx->SetArrayResult ( true );
$res = $sphinx->Query($_GET['key'],'*');
var_dump($res);
```

#### **thinkphp5 使用介绍**

1.将`sphinxapi.php`文件放到`extend`目录下

2.在控制器方法中使用（`app/api/index`）

```php
public function test()
{
    $sphinx = new \SphinxClient();
    // sphinx的主机名和端口
    $sphinx->SetServer('localhost',9312);
    // 匹配模式
    $sphinx->SetMatchMode(SPH_MATCH_ANY);
    // 设置返回结果集为php数组格式

    $sphinx->SetArrayResult ( true );
    $res = $sphinx->Query(input('key'),'*');
    var_dump($res);
}
```

3.url 访问：
`http://localhost/mypro/api/index/test?key=test`

4.输出数据

```json
D:\web\COD\api\application\api\controller\Index.php:21:
array (size=10)
  'error' => string '' (length=0)
  'warning' => string '' (length=0)
  'status' => int 0
  'fields' =>
    // 查询显示的字段名
    array (size=2)
      0 => string 'title' (length=5)
      1 => string 'content' (length=7)
  'attrs' =>
    array (size=2)
      'group_id' => string '1' (length=1)
      'date_added' => string '2' (length=1)
  'matches' =>
    // 匹配的结果，返回匹配记录的id和权重（权重越大，匹配条件越多）
    array (size=3)
      0 =>
        array (size=3)
          'id' => string '1' (length=1)
          'weight' => int 2421
          'attrs' =>
            array (size=2)
              ...
      1 =>
        array (size=3)
          'id' => string '2' (length=1)
          'weight' => int 2421
          'attrs' =>
            array (size=2)
              ...
      2 =>
        array (size=3)
          'id' => string '4' (length=1)
          'weight' => int 1442
          'attrs' =>
            array (size=2)
              ...
  'total' => int 3
  'total_found' => int 3
  'time' => float 0
  'words' =>
    array (size=1)
      'test' =>
        array (size=2)
          'docs' => int 6
          'hits' => int 10
```

## 四、在 ThinkPHP5 项目中应用

### **1.修改配置信息**`sphinx/bin/sphinx.conf`

```ini
source src1{
# 省略其他配置
sql_user  = root
sql_pass  = 123123
sql_db    = shopMall
sql_query = \
  SELECT id,offerid, name, ename, setmeal, category, country, traffic, os, body, inventory_title, shop \
  FROM i_offer
sql_attr_uint = offerid
# 省略其他配置
}
```

```ini
index rt
{
  type  = rt
  path  = D:\Service\sphinx\data\rt

  rt_field  = name
  rt_field  = ename
  rt_field  = setmeal
  rt_field  = category
  rt_field  = country
  rt_field  = traffic
  rt_field  = body

  rt_attr_uint  = offerid
}
```

### **2.生成索引，并开启 searchd 服务**

```bash
# 生成项目索引
sphinx/bin/indexer.exe --config sphinx.conf --all

# 开启服务 &表示后台开启，不用保持窗口执行状态
sphinx/bin/searchd.exe &
```

### **3.程序实现**

sphinx 查询返回的结果并不是我们需要的显示结果，所以还需要对结果进行处理，从而获取到我们需要的结果。

默认 sphinx 返回的数据中包含 id 信息是和数据记录的信息是相关的，所以我们需要通过 id 到数据库中查询相关信息。

```php
public function test()
{
    $s = new \SphinxClient;
    $s->setServer("localhost", 9312);
    // 作为数组返回
    $s->SetArrayResult(true);
    // 匹配格式  任意匹配
    $s->setMatchMode(SPH_MATCH_ANY);
    $s->setMaxQueryTime(3);
    // input()表示接收用户传过来的数据
    $result = $s->query(input('key'),'*');

    return json($result);
}
```

### **4.测试实现**

访问 url：
`http://localhost/mypro/api/index/test?key=官方`

返回结果：

```json
D:\web\COD\api\application\api\controller\Index.php:22:
array (size=10)
  'error' => string '' (length=0)
  'warning' => string '' (length=0)
  'status' => int 0
  'fields' =>
    array (size=10)
      0 => string 'name' (length=4)
      1 => string 'ename' (length=5)
      2 => string 'setmeal' (length=7)
      3 => string 'category' (length=8)
      4 => string 'country' (length=7)
      5 => string 'traffic' (length=7)
      6 => string 'os' (length=2)
      7 => string 'body' (length=4)
      8 => string 'inventory_title' (length=15)
      9 => string 'shop' (length=4)
  'attrs' =>
    array (size=1)
      'offerid' => string '1' (length=1)
  'matches' =>
    array (size=6)
      0 =>
        array (size=3)
          'id' => string '36' (length=2)
          'weight' => int 4667
          'attrs' =>
            array (size=1)
              ...
      1 =>
        array (size=3)
          'id' => string '19' (length=2)
          'weight' => int 2611
          'attrs' =>
            array (size=1)
              ...
      // 此处省略部分数据
  'total' => int 6
  'total_found' => int 6
  'time' => float 0
  'words' =>
    array (size=2)
      '官' =>
        array (size=2)
          'docs' => int 14
          'hits' => int 16
      '方' =>
        array (size=2)
          'docs' => int 70
          'hits' => int 94
```

对结果进行处理，支持过滤查询`SetFilter()`。

```php
public function test()
{
    $filterkey = 'status';
    $filtervalue = "1";

    $sphinx = new \SphinxClient;
    $sphinx->setServer("localhost", 9312);

    // 进行过滤查询
    if (!empty($filterkey)) {
        // 只在status==1的记录中检索
        $filterRes = $sphinx->SetFilter($filterkey, [$filtervalue]);
        if (!$filterRes) {
            return false;
        }
    }

    // 作为数组返回
    $sphinx->SetArrayResult(true);
    // 匹配格式  任意匹配
    $sphinx->setMatchMode(SPH_MATCH_ANY);
    $sphinx->setMaxQueryTime(3);
    // input()表示接收用户传过来的数据
    $result = $sphinx->query(input('key'),'*');

    // 避免没有匹配记录时报错
    if(empty($result['matches'])) {
        return null;
    }

    $result = $result['matches'];
    // 返回数组中指定的id列, 返回结果为单列数组
    $result = array_column($result, 'id');
    $list = model('offer')
    ->field('offerid, status, name, ename, setmeal, category, country, traffic, os, body, inventory_title, shop')
    ->where(array('id' => array('in', $result)))
    ->select();

    return json($list);
}
```

返回结果

```json
[
  {
    offerid: 2332302,
    status: 1,
    name: "【官方站】減震隱形增高鞋墊（安全有效~秒增高5公分~）",
    ename: "zenggaoxiedian",
    setmeal: "日韓超夯，氣墊隱形增高墊，輕鬆增高5公分，透氣減震，抗菌防臭，藝人最愛！【可拆分，自由裁剪，均碼35-44可用】【超殺998三雙】",
    category: "[{"id":6,"name":"其他"},{"id":7,"name":"商城"},{"id":8,"name":"家庭用品\n"}]",
    country: "[{"id":11,"name":"American Samoa"},{"id":1,"name":"Andorra"},{"id":8,"name":"Angola"},{"id":5,"name":"Anguilla"},{"id":10,"name":"Argentina"},{"id":7,"name":"Armenia"},{"id":12,"name":"Austria"}]",
    traffic: "[{"id":2,"name":"16+"},{"id":3,"name":"3G\/4G"},{"id":4,"name":"Adult"}]",
    os: "[{"id":1,"name":"3DS System Software"},{"id":2,"name":"Android"},{"id":13,"name":"BeOS"},{"id":16,"name":"CentOS"}]",
    body: "123123123",
    inventory_title: "隱形增高鞋墊B",
    shop: "[{"userid":77912776,"name":"myShop"}]"
  },
  {
    offerid: 3308032,
    status: 1,
    name: "【官方站】電熱造型梳",
    ename: "zaoxingshu",
    setmeal: "長/短髮都適用！好梳好上手！亂翹髮尾一秒聽話！【人氣爆红款美髮救星】限時特價，加NT$300再得1件！！！",
    category: "[{"id":2,"name":"美容"},{"id":4,"name":"日用品"},{"id":6,"name":"其他"},{"id":8,"name":"家庭用品\n"}]",
    country: "[{"id":6,"name":"Albania"},{"id":4,"name":"Antigua And Barbuda"}]",
    traffic: "[{"id":3,"name":"3G\/4G"},{"id":5,"name":"Adult Content"},{"id":6,"name":"App Discovery Traffic"}]",
    os: "[{"id":3,"name":"Android with AOKP"},{"id":5,"name":"Android with Cyanogen Mod"},{"id":6,"name":"Android with LiquidSmooth"},{"id":7,"name":"Android with MIUI"}]",
    body: "123123123"
    inventory_title: "NOVA多功能卷髮棒B",
    shop: "[{"userid":77912776,"name":"myShop"}]"
  }
]
```

## 五、扩展补充

需要对多个不同表创建不同的检索的使用方法：

只需创建不同的索引源和索引，相应的对应好即可。

实际项目中的sphinx完整配置：

```ini
source keyword
{
  type      = mysql
  sql_host  = localhost
  sql_user  = root
  sql_pass  = 123
  sql_db    = goods
  sql_query_pre   = SET NAMES utf8

  sql_query  = \
    SELECT id,status,keyword \
    FROM t_keyword

  # 设置查询条件
  sql_attr_uint = status
}

index keyword
{
  source  = keyword
  path  = D:/UPUPW_ANK_W64/Database/data_sphinx/keyword
  charset_type  = utf-8
  charset_table = 0..9, A..Z->a..z, _, a..z, U+410..U+42F->U+430..U+44F, U+430..U+44F
  ngram_len = 1
  ngram_chars = U+3000..U+2FA1F
}

source collection
{
  type        = mysql
  sql_host    = localhost
  sql_user    = root
  sql_pass    = 123456
  sql_db      = goods
  sql_query   = \
    SELECT id, status, title, offerid, keyword, searchid, fromname \
    FROM t_collection
}

index collection
{
  source    = collection
  path      = D:/UPUPW_ANK_W64/Database/data_sphinx/collection
  mlock     = 0
  morphology      = none
  min_word_len    = 1
  charset_table   = 0..9, A..Z->a..z, _, a..z, U+410..U+42F->U+430..U+44F, U+430..U+44F
  min_prefix_len  = 0
  min_infix_len   = 1
  ngram_len   = 1
  ngram_chars   = U+3000..U+2FA1F
  html_strip    = 0
}

source goods
{
  type        = mysql
  sql_host    = localhost
  sql_user    = root
  sql_pass    = 123455
  sql_db      = goods
  sql_query   = \
    SELECT id, status, name, seller_region, keywords, offerid, searchid, detailid, keyword, fromname \
    FROM t_goods
}

index goods
{
  source    = goods
  path      = D:/UPUPW_ANK_W64/Database/data_sphinx/goods
  mlock     = 0
  morphology      = none
  min_word_len    = 1
  charset_table   = 0..9, A..Z->a..z, _, a..z, U+410..U+42F->U+430..U+44F, U+430..U+44F
  min_prefix_len  = 0
  min_infix_len   = 1
  ngram_len       = 1
  ngram_chars     = U+3000..U+2FA1F
  html_strip      = 0
}

indexer
{
  mem_limit   = 128M
}

searchd
{
  listen    = 9312
  #listen   = 9306:mysql41
  log   = D:/UPUPW_ANK_W64/Logs/sphinx.log
  query_log       = D:/UPUPW_ANK_W64/Modules/Sphinx/log/query.log
  read_timeout    = 5
  client_timeout  = 300
  max_children    = 30
  persistent_connections_limit  = 30
  pid_file        = D:/UPUPW_ANK_W64/Modules/Sphinx/log/searchd.pid
  seamless_rotate = 1
  preopen_indexes = 1
  unlink_old      = 1
  max_packet_size = 8M
  max_filters     = 256
  max_filter_values = 4096
  max_batch_queries = 32
}
```


## 六、参考资料：

> PHP 官方手册使用 Sphinx 介绍：

http://www.php.net/manual/zh/book.sphinx.php

> sphinx 安装：

https://blog.csdn.net/huang2017/article/details/69665057

https://blog.csdn.net/huang2017/article/details/69666154

> 将 sphinx 服务添加到 windows 服务：

`./searchd.exe --install -c sphinx.conf --servicename s`

https://blog.csdn.net/design321/article/details/8895712

> sphinx 使用：

https://blog.csdn.net/u010837612/article/details/70827481

> 中文支持（linux 系统）

http://www.xuejiehome.com/blread-1283.html

> 中文支持（windows 系统）--- 默认已能支持中文

http://www.phpernote.com/php-template-framework/284.html

> 其他

https://my.oschina.net/guyson/blog/283576

[下载]: http://sphinxsearch.com/downloads/release/
[前往]: https://pecl.php.net/package/sphinx/1.3.2/windows
