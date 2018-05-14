---
title: sphinx-php-windows
date: 2018-05-14 09:41:08
tags:
    - sphinx
    - windows
    - php
category: 
    - PHP
    - Sphinx
---

## 安装（Windows）

**1.官方下载**

下载地址： 下载[1]

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

(1)将`sphinx/etc/sphinx.conf.dist`文件复制到`sphinx/bin/`目录下，并重命名为`sphinx.conf`

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
```

> source src1throttled : src1{}

```ini
# 主要需要修改的配置项，默认为/var/data/test1
path = D:\Service\sphinx\data\test1
```

> index test1stemmed : test1{}

```ini
# 主要需要修改的配置项，默认为/var/data/test1stemmed

path = D:\Service\sphinx\data\test1stemmed
```

> index rt{}

```ini
# # 主要需要修改的配置项，默认为/var/data/rt
path = D:\Service\sphinx\data\rt
```

> searchd{}

```ini
# 自定义日志文件位置
log = D:\Service\sphinx\log\searchd.log
query_log = D:\Service\sphinx\log\query.log
pid_file = D:\Service\sphinx\log\searchd.pid
```

以下几项不需要修改默认值，即可直接使用
> index test1{}

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
indexer.exe --config sphinx.conf --all

# 启用搜索服务
searchd.exe --pidfile
```

## PHP打开sphinx扩展
**1.下载扩展**: 下载[2]

具体需要下载的版本需要查看phpinfo信息，

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
|id  | group_id |  group_id2 |  date_added| title|
|------| -------:|-------:|------:|:------:|
|1|	1|	5|	2018-05-14 09:12:25|	test one|	this is my test document number one. also checking search within phrases.|
|2|	1|	6|	2018-05-14 09:12:25|	test two|	this is my test document number two|
|3|	2|	7|	2018-05-14 09:12:25|	another doc|	this is another group|
|4|	2|	8|	2018-05-14 09:12:25|	doc number four|	this is to test groups|
```

2.PHP代码实现

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
    $sphinx->SetServer('localhost',9312);
    $sphinx->SetMatchMode(SPH_MATCH_ANY);
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
    array (size=2)
      0 => string 'title' (length=5)
      1 => string 'content' (length=7)
  'attrs' => 
    array (size=2)
      'group_id' => string '1' (length=1)
      'date_added' => string '2' (length=1)
  'matches' => 
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



[1]: http://sphinxsearch.com/downloads/release/
[2]: https://pecl.php.net/package/sphinx/1.3.2/windows 