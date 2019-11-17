---
title: 【TP5深入理解】控制器(三)--前置操作
date: 2018-08-07 20:26:33
tags:
  - thinkphp5
category:
  - 【PHP框架】
---

tp5 框架的前置操作可以用于对某些方法进行通用的预处理，比如登录状态的判断[session 处理]、用户权限的卡控[cache/session 的处理]，通过控制器的前置操作，将公用代码进行封装，简化了调用流程[直接设定前置关系即可实现前置方法的自动调用]。

<!--more-->

## 基本用法

### 使用示例：

```php
protected $beforeActionList = [
    'userBeforeAction1' => ['only' => 'function1NeedDoBeforeAction1, function2DoNeedBeforeAction1'],
    'userBeforeAction2' => ['except' => 'function1NotNeedBeforeAction2, function2NotNeedBeforeAction2'],
    'userBeforeAction3'
];
```

### 使用说明：

(1) `only` => 当调用 api 接口方法`function1NeedDoBeforeAction1()`和`function1NeedDoBeforeAction1`时，都会自动调用不公开[`protected`]的前置方法`userBeforeAction1()`，并且该前置方法只在访问这两个 api 方法时执行。

(2) `except` => api 接口方法`function1NotNeedBeforeAction2()`和`function1NotNeedBeforeAction2()`时，不会执行前置方法`userBeforeAction2()`

## 使用介绍

> 可以为某个或者某些操作指定前置执行的操作方法，设置 beforeActionList 属性可以指定某个方法为其他方法的前置操作，数组键名为需要调用的前置方法名，无值的话为当前控制器下所有方法的前置方法。

### 控制器类属性: $beforeActionList

### 属性值： 键值对

- 键：前置方法名
- 值：前置方法作用域[键值对|无]

  - 无值时： 对当前控制器所有 api 方法都执行前置
  - 有值[键值对]：

    - 键：
      - except：除某些 api 方法执行前置
      - only：只对某些 api 方法执行前置
    - 值：需要进行前置操作的 api 方法 [当该项值为多个时，用半角,进行间隔]

### 【注意点】：

（1）由于 TP5 框架对 url 的处理是全部转化为小写，并且执行前置操作时，也是通过 url 中参数获取当前调用的 api 方法，并判断是否需要进行前置操作的，所以定义需要执行前置方法的 api 方法时，都需要使用小写[针对 TP5.0 版本]

（2）前置方法的访问方式为`private`时，则无法调用。

## 实现原理[源码阅读]

### 框架类库

基类控制器`thinkphp\library\think\Controller.php`

### 相关介绍

(1) 属性名：`$beforeActionList`

前置方法列表：

```php
protected $beforeActionList = [];
```

(2) 构造方法：`__construc()`

```php
// 遍历前置方法列表，并对每个前置方法进行前置处理
if ($this->beforeActionList) {
    foreach ($this->beforeActionList as $method => $options) {
        is_numeric($method) ?
        $this->beforeAction($options) :
        $this->beforeAction($method, $options);
    }
}
```

【逻辑分析】：

- 判断前置数组的键是否为数值

- 如果是数值，直接对前置方法键值对的值进行处理

- 如果不是数值，则对每组前置关系进行处理

### 实现方法：`beforeAction()`

(1) 当**前置方法**[每组前置关系的键]是数值时 【键为数值的情况即 **该组前置关系** 没有指定前置方法的作用域，是对所有方法执行该前置方法】

- 调用 -- `$this->beforeAction($options)`

- 实际执行过程为

```php
protected function beforeAction($options)
{
    call_user_func([$this, $options]);
}
```

(2) 当**前置方法**[每组前置关系的键]不是数值时

- 调用 -- `$this->beforeAction($method, $options)`

- 完整执行过程为

```php
protected function beforeAction($method, $options = [])
{
    if (isset($options['only'])) {
        if (is_string($options['only'])) {
            $options['only'] = explode(',', $options['only']);
        }

        if (!in_array($this->request->action(), $options['only'])) {
            return;
        }
    } elseif (isset($options['except'])) {
        if (is_string($options['except'])) {
            $options['except'] = explode(',', $options['except']);
        }

        if (in_array($this->request->action(), $options['except'])) {
            return;
        }
    }

    call_user_func([$this, $method]);
}
```

（3）函数 [[call_user_func](http://php.net/manual/en/function.call-user-func.php)]功能：把第一个参数作为回调函数调用

- 参数说明：

  当传入的参数为一个数组时，则将数组的

  - 第一个元素作为 **类名** 或 **类的实例化对象**
  - 第二个元素作为类的方法名 [一般方法和静态方法均可以]

- 调用结果：

  就是直接执行该类下的相应方法的结果

[全程学习+笔记时长：100min]

【声明】

我的博客即将搬运同步至腾讯云+社区，邀请大家一同入驻：[https://cloud.tencent.com/developer/support-plan?invite_code=89fda9dsh3d0](https://cloud.tencent.com/developer/support-plan?invite_code=89fda9dsh3d0)
