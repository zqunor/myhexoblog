---
title: windows7使用Sphinx+PHP+MySQL详细介绍
date: 2018-05-14 09:41:08
tags:
    - sphinx
    - windows
    - php
    - mysql
category:
    - Server
    - PHP
    - Sphinx
---

由于业务需要，需要做类似淘宝商城商品检索的功能，对于数据量很大的情况，MySQL查询的效率损耗很大，需要使用专门的索引引擎进行搜索查询，实现功能，对于和PHP和Mysql的结合的索引引擎中， xunsearch和sphinx是较为著名的，但由于xunsearch服务器端不支持windows，所以暂且先考虑sphinx的使用。

<!--more-->

## 安装（Windows）

**1.官方下载**

Sphinx下载地址： [下载]

**2.解压并重命名**

此处下载版本为`3.0.3`，将 sphinx 文件夹命名为`sphinx`

**3.文件夹目录介绍**

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

**4.设置配置文件**

(1)将`sphinx/etc/sphinx-min.conf.dist`文件复制到`sphinx/bin/`目录下，并重命名为`sphinx.conf`

注：`sphinx/etc/sphinx.conf.dist`为带注释的详细的

(2)设置配置项

主要是以下为配置函数：

> source src1{} --- 连接数据库的基本配置

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
sql_query		= \
		SELECT id, group_id, UNIX_TIMESTAMP(date_added) AS date_added, title, content \
		FROM documents
```

> index test1{}
```ini
# 索引数据存放目录，默认为/var/data/test1
path = D:\Service\sphinx\data\test1

# 设置中文匹配
min_word_len    = 1  
charset_type    = utf-8  
# 指定utf-8的编码表
charset_table   = 0..9, A..Z->a..z, _, a..z, U+410..U+42F->U+430..U+44F, U+430..U+44F
min_word_len	= 1
min_prefix_len	= 0
min_infix_len	= 1

# 开启中文分词支持
ngram_len		= 1
# 需要分词的字符
ngram_chars		= U+3000..U+2FA1F
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

# 
rt_attr_uint	= offerid
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

> index dist1{}

> indexer{}

> common{}

**5.操作数据库，导入样例数据**

(1)进入到mysql命令行，执行命令
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

**6.生成索引文件**

cmd命令行进入到`sphinx/bin/`目录下
```bash
# 生成索引文件
> indexer.exe --config sphinx.conf --all
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

**7.开启搜索服务，保持后台运行**
```bash
> searchd.exe --pidfile

[Tue May 15 09:02:14.690 2018] [7776] using config file './sphinx.conf'...
listening on all interfaces, port=9312
listening on all interfaces, port=9306
Sphinx 3.0.3-dev (commit facc3fb)
Copyright (c) 2001-2018, Andrew Aksyonoff
Copyright (c) 2008-2016, Sphinx Technologies Inc (http://sphinxsearch.com)

```

## PHP开启sphinx扩展
**1.下载php_sphinx扩展**: [前往]

具体需要下载的版本需要查看phpinfo信息:

    Architecture         ==》x86/x64
    PHP Extension Build  ==》NTS/NS

下载并解压后，将`php_sphinx.dll`文件放到php/ext目录下

**2.修改php.ini配置文件**
```ini
# 在 Dynamic Extensions 列表中添加php_sphinx扩展
extension=php_sphinx.dll
```

修改后重启apache服务

**3.在phpinfo.php输出的信息中查看sphinx扩展是否安装成功**
```info
            sphinx
sphinx support	 enabled
Version	         1.3.2
Revision	 $Revision$
```


## 代码实现
1.样例数据表`test.documents`记录：
```info
| id  | group_id |  group_id2 |  date_added | title |
| ----- | -------: | -------: | ------: | :------: |
| 1 |	1 |	5 |	2018-05-14 09:12:25 |	test one |	this is my test document number one. also checking search within phrases. |
| 2 | 1 |	6 |	2018-05-14 09:12:25 |	test two |	this is my test document number two |
| 3 |	2 |	7 |	2018-05-14 09:12:25 |	another doc |	this is another group|
| 4 |	2 |	8 |	2018-05-14 09:12:25 |	doc number four |	this is to test groups |
```

2.PHP代码实现

**一般实现**
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

**thinkphp5使用介绍**

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

3.url访问：
`http://localhost/api/index/test?key=test`

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

## 在ThinkPHP5项目中应用
1.修改配置信息




在项目中应用

sphinx查询返回的结果并不是我们需要的显示结果，所以还需要对结果进行处理，从而获取到我们需要的结果。

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
    $result = $s->query(input('key'),'*');

    // 避免没有匹配记录时报错
    if(empty($result['matches'])) {
        return null;
    }

    $result = $result['matches'];
    // 返回数组中指定的id列, 返回结果为单列数组
    $result = array_column($result, 'id');
    $list = model('offer')
    ->field('offerid, name, ename, setmeal, category, country, traffic, os, body, inventory_title, shop')
    ->where(array('id' => array('in', $result)))
    ->select();

    return json($list);
}
```


参考连接：
> PHP官方手册使用Sphinx介绍：

http://www.php.net/manual/zh/book.sphinx.php

> sphinx安装：

https://blog.csdn.net/huang2017/article/details/69665057

https://blog.csdn.net/huang2017/article/details/69666154

> 将sphinx服务添加到windows服务：

https://blog.csdn.net/design321/article/details/8895712

> sphinx使用：

https://blog.csdn.net/u010837612/article/details/70827481

> 中文支持（linux系统）

http://www.xuejiehome.com/blread-1283.html

> 中文支持（windows系统）

http://www.phpernote.com/php-template-framework/284.html

> 其他
https://my.oschina.net/guyson/blog/283576

[下载]: http://sphinxsearch.com/downloads/release/
[前往]: https://pecl.php.net/package/sphinx/1.3.2/windows 