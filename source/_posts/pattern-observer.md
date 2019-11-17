---
title: 【设计模式】观察者模式
date: 2019-05-7 00:02:15
tags:
  - 设计模式
category:
  - 【设计模式】
---

## 基本介绍

1、观察者模式（Observer）：当一个对象状态发生改变时，依赖它的对象全部会收到通知，并自动更新

2、场景： 当一个事件发生后，要执行一连串更新操作。传统的编程方式，就是在事件的代码之后直接加入处理逻辑。当更新的逻辑增多之后，代码会变得难以维护。这种方式是耦合的，侵入式的，增加新的逻辑需要修改事件主体的代码。

3、观察者模式实现了低耦合，非侵入式的通知与更新机制。

<!-- more -->

## 简单代码实现

### （抽象）目标
定义一个观察者集合，并提供方法来增加和删除观察者对象，还包括通知方法
```php
abstract class Subject
{
    private $observers = [];
    function addObserver(Observer $observer)
    {
        $this->observers[] = $observer;
    }

    function notify()
    {
        foreach ($this->observers as $observer) {
            $observer->update();
        }
    }
}
```

### 具体目标

```php
class ConcreteSubject extends Subject
{
    function trigger()
    {
        echo "本身的动作 <br />\n";
    }
}
```

### （抽象）观察者

对观察目标的改变做出反应，只申明更新数据的方法

```php
interface Observer
{
    function update($event_info = null);
}
```

### 具体观察者

观察到目标变化后需要执行的具体动作。可添加到目标类的观察者集合中，或从目标类的观察者集合中删除

具体动作1：
```php
class Observer1 implement Observer
{
    function update($event_info = null)
    {
        echo "逻辑1 <br />\n";
    }
}
```

具体动作2：
```php
class Observer2 implement Observer
{
    function update($event_info = null)
    {
        echo "逻辑2 <br />\n";
    }
}
```

### 调用与实现

```php
$event = new Event;
$event->addObserver(new Observer1);
$event->addObserver(new Observer2);
$event->trigger();
```

## 扩展

实际情况中，具体观察者类的update()方法在执行时需要使用到具体目标类中的状态或属性。

所以，具体观察者和具体目标类之间有时候还需要存在关联或依赖关系。在具体观察者类中定义一个具体目标实例，通过该实例获取存储在具体目标类中的状态或属性值。

## 举例

> 订单流程中，当订单审核拒绝时，除了更新**主订单表**中的状态，可能还需要添加一条拒绝理由的**订单备注表**的记录，以及**状态变更日志表**的记录。

此时，

**订单对象**是具体目标，`更新主订单表中的状态`是具体目标需要执行的动作，

**订单备注表**就是观察者1，它执行的动作是`添加一条拒绝理由表记录`，

**状态变更日志表**也是观察者2，它执行的动作是`添加状态变更日志表记录`

<br/>

但是，

观察者1和观察者2往往需要用到具体目标的订单编号和状态值，此时如何进行参数传递？

目前想到的方案：
- 通过 `$event_info` 参数
- 在抽象目标中执行notify的时候，可以传递一个update的参数，使其能在具体观察者中使用。

## 参考资料

- [慕课网 - 大话设计模式](https://www.imooc.com/video/5037)

- [史上最全设计模式导学目录（完整版）](https://blog.csdn.net/LoveLion/article/details/17517213)