---
title: 【实战】tp5+小程序(一)--数据库访问与ORM
date: 2018-07-1 16:26:58
tags:
  - thinkphp5
  - 微信小程序
category:
  - 【PHP框架】
---

ThinkPHP5 从入门到深入学习，结合实战项目深入理解 ThinkPHP5 的特性和使用方法，了解 ThinkPHP5 的数据库访问和 ORM 思想，学习使用查询构造器、日志以及初步了解异常处理机制。

<!--more-->

## 7.2 从一个错误了解 Exception 的继承关系

1.将`Exception`修改为全局`Exception`基类，而不是`think\Exception`

    `think\Exception => \Exception =>  Throwable`
    `HttpException => \RuntimeException =>  \Exception =>  Throwable`

**当访问的控制器不存在、url 错误时，属于 HttpException 异常。** 而原先定义的`render()`和`recordErrorLog()`方法要求接收的参数类型是`think\Exception`，由于`HttpException`不是继承于`think\Exception`，不能转化为`think\Exception`，所以会报错。

> 解决：使用全局 Exception 基类后，既支持 HttpException，又支持 think\Exception。子类可以自动被转化为父类的类型。
> Exception 基类是所有异常类的基类。

2.**补充**：PHPStrom 快捷键：

- ctrl+alt+O => 快速删除没有 use 的类

## 7.3 TP5 数据库中间层架构解析

![tp5数据库架构](https://ws1.sinaimg.cn/large/005EgYNMgy1fucheex8nsj30g10cxt9l.jpg)

## 7.4 查询构造器一

1、为什么不使用原生的查询语句而使用查询构造器？

- 简洁方便
- 对不同数据库的操作进行了封装，使用统一的数据库操作标准

2、对查询构造器的理解

- 查询构造器不仅仅是对数据库进行读操作，还包括数据库的写操作
- 查询构造器仅仅是语法，最终都是通过 Builder 翻译成 SQL 语句来执行

## 7.5 查询构造器二

1.查询语句 = 链式方法 + 执行方法

2.链式方法：

- where
- whereOr
- field
- ...

只会返回 Query 对象，不是查询结果

3.执行方法：

- find
- select
- update
- delete
- insert

  4.**在执行方法调用前，查询状态是保留的，直到调用执行方法后，状态才会被清除**

## 7.6 查询构造器三

1.链式方法说明（where）：

> where('字段名','表达式','查询条件')

2.三种实现方式：

- 表达式

- 数组法（不够灵活，且存在一定的安全问题）

- 闭包（最灵活）

```php
// 通过use来使用外部的数据
where(function ($query) use ($id){
    $query->where('banner_id', '=', $id)
})
```

## 7.7 开启 SQL 日志记录

1.database.php =》 debug=true

2.config.php =》 app_debug=true

3.config.php =》 log =》 level =》sql

```php
'log'                    => [
     日志记录方式，内置 file socket 支持扩展
    'type'  => 'File',
     日志保存目录
    'path'  => LOG_PATH,
     日志记录级别
    'level' => ['error', 'sql'],
],
```

不在配置文件中开启日志

`ExceptionHandler`=>`recordErrorLog` 方法中开启日志仅仅在发生异常时写入。

所以对于全局的情况，将日志手动添加到入口文件中，对所有调用都开启 sql 日志

？？这样和在配置文件中开启日志有什么区别？

## 7-8 ORM 与模型

1.ORM 理解：

ORM = `Object Relationship Map`

对象关系映射:将每张数据表看作是一个对象

2.模型（TP5 模型）--ORM 实现的具体机制
=> `业务的集合= 数据库查询+业务逻辑+...`

【注】模型与数据表不是一一对应的关系，简单的业务逻辑看上去是一张表对应一个模型，但复杂的业务逻辑（需要分层）可能是横跨多个表。

模型不仅仅只是 model 这一层，复杂的业务还可以继续划分，tp5 中有 `model`(数据层，细)，`service`(服务层，粗)，`logic`(逻辑层)

## 7-9 初识模型

1.`model/Banner.php` 继承 Model 类之后，就成为了模型，就可以使用模型类已经封装好的方法(`get`)，而不用自定义方法(`getBannerById`)。

```php
$banner = BannerModel::getBannerById($id);
 // => 返回数组
 //等价于
$banner = BannerModel::get($id);
// => 返回对象，便于处理查询结果
```

2.tp5 自动将对象序列化，序列化的格式根据配置文件中的配置项`default_return_type`转化为相应的格式。

```php
// config.php
'default_return_type' => 'html'
```

3.自定义模型方法(`getBannerById`)调用的是指定的数据表，而模型类自动的方法(`get`)调用的是模型类名对应的数据表。
