---
title: 【设计模式】适配器模式
date: 2019-04-28 00:03:57
tags:
  - 设计模式
category:
  - 【设计模式】
---

适配器模式(Adapter Pattern)：将一个接口转换成客户希望的另一个接口，使接口不兼容的那些类可以一起工作，其别名为包装器(Wrapper)。适配器模式既可以作为类结构型模式，也可以作为对象结构型模式。

<!-- more -->

## 应用：数据库适配器

1、定义一个统一方法的数据库接口（目标接口/目标抽象类）

```php
interface IDatabase
{
    // 数据库连接
    function connect($host, $user, $passwd, $dbname);
    // 执行sql
    function query($sql);
    // 关闭数据库连接
    function close();
}
```

2、单个数据库类定义

(1) MySQL  （mysql数据库操作适配器类）
- 类文件
```php
class MySQL implements IDatabase
{
    protected $conn;
    function connect($host, $user, $passwd, $dbname)
    {
        $conn = mysql_connect($host, $user, $passwd);
        mysql_select_db($dbname, $conn);
        $this->conn = $conn;
    }

    function query($sql)
    {
        return mysql_query($sql, $this->conn);
    }

    function close()
    {
        mysql_close($this->conn);
    }
}
```

- 单一调用

```php
$db = new MySQL();
$db->connect('127.0.0.1', 'root', 'root', 'test');
$db->query('show databases');
$db->close();
```

(2) MySQLi  （mysqli数据库操作适配器类）
- 类文件定义
```php
class MySQL implements IDatabase
{
    protected $conn;
    function connect($host, $user, $passwd, $dbname)
    {
        $conn = mysqli_connect($host, $user, $passwd,  $dbname);
        $this->conn = $conn;
    }

    function query($sql)
    {
        return mysqli_connect($this->conn, $sql);
    }

    function close()
    {
        mysql_close($this->conn);
    }
}
```

(3) PDO （PDO数据库操作适配器类）
- 类文件定义
```php
class PDO implements IDatabase
{
    protected $conn;
    function connect($host, $user, $passwd, $dbname)
    {
        $conn = new \PDO("mysql:host={$host};dbname={$dbname}",$username, $password);
        $this->conn = $conn;
    }

    function query($sql)
    {
        return $this->conn->query($sql);
    }

    function close()
    {
        unset($this->conn);
    }
}
```

3、理解

多个类文件定义完成后，需要使用数据库的时候，直接通过类名切换实现不同数据库类的调用

mysql、mysqli、PDO数据库操作类本身的库为适配者

被实现的接口为目标接口

封装的三个数据库操作类为三个适配器

```php
$db = new MySQL();
//$db = new MySQLi();
//$db = new PDO();
$db->connect('127.0.0.1', 'root', 'root', 'test');
$db->query('show databases');
$db->close();
```