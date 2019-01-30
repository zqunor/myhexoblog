---
title: 【算法】PHP实现全排列
date: 2018-06-09 13:50:55
tags:
    - 算法
    - 全排列
category:
    - Algorithm
    - Permutation
---
在购物网站中，商品的规格往往有多种，而不同规格的不同属性之间可以有不同的组合方式，而每一种组合方式都是以一个整体的形式存在，所以需要通过算法中的全排列，对不同规格下的属性进行全排列。

<!--more-->

## 一、需求介绍

### 1.当前的规格形式（多种规格，且每种规格下都有多种子规格）

```json
[
    {
        "attr_name": "颜色",
        "value_name": [
            "黄色",
            "绿色",
            "黑色"
        ]
    },
    {
         "attr_name": "尺码",
        "value_name": [
            "S",
            "M",
            "L",
            "XL"
        ]
    },
        {
        "attr_name": "赠品",
        "value_name": [
            "苹果",
            "香蕉",
            "梨子"
        ]
    }
]
```

### 2.需要转换成的样式

三种规格组合在一起决定了一个具体的商品。每一种规格属性都可能和其他规格下的属性值进行组合，所以共有`3*4*3=36`种可能。

```json
[
    {
        {
            "attr_name": "颜色",
            "value_name": "黄色"
        },
        {
            "attr_name": "尺码",
            "value_name": "S"
        },
        {
            "attr_name": "赠品",
            "value_name": "苹果"
        }
    },
    {
        {
            "attr_name": "颜色",
            "value_name": "黄色"
        },
        {
            "attr_name": "尺码",
            "value_name": "S"
        },
        {
            "attr_name": "赠品",
            "value_name": "香蕉"
        }
    },
    {...}
]
```

## 二、算法思想

### 1.简单思路

(1)先获取第一种规格的名称和其中一个属性值，再获取第一种规格的名称和其中一个属性值，组成数组，..., 以此类推，直到所有规格的最后一个属性值组成一个数组。

(2)在例举的案例中，第一种规格有三个属性，第二种规格有四个属性，第三种规格有三个属性，即组合的可能情况为`3*4*3=36`种情况。

(3)较为简单的理解方法是对所有规格进行循环，类似冒泡排序，但这种方法对规格少的情况可能实现较为方便，但是一旦规格多了，循环的层级规模将会很大，将大大影响性能。

(4)可以考虑**栈**的数据结构和`while`循环实现。

### 2.进一步延伸

(1)php中栈的实现形式: `array_pop()`

(2)

## 三、代码实现

### 1.功能函数介绍

(1)array_pop() -- 弹出最后一个单元（出栈），并将数组的长度减一

* 具体使用：`mixed array_pop ( array &$array )`

弹出并返回 array 数组的最后一个单元，并将数组 array 的长度减一。

> Note: 使用此函数后会重置（reset()）array 指针。

* 返回值：返回 array 的最后一个值。如果 array 是空（如果不是一个数组），将会返回 NULL 。

(2)array_merge()

* 具体使用：`array array_merge ( array $array1 [, array $... ] )`

将一个或多个数组的单元合并起来，一个数组中的值附加在前一个数组的后面。返回作为结果的数组。

> Note:
>* 如果输入的数组中有相同的字符串键名，则该键名后面的值将覆盖前一个值。然而，如果数组包含数字键名，后面的值将不会覆盖原来的值，而是附加到后面。
>* 如果只给了一个数组并且该数组是数字索引的，则键名会以连续方式重新索引。

* 返回值：返回合并后的数组。

### 2.完整代码实现

```php
function goodsSpecPermutation($specs)
{
    // 获取第一个规格的信息
    $spec = array_pop($specs);
    $attr = $spec['attr_name'];
    $valueArray = $spec['value_name'];

    while ($specs) {
        $mergeArray = array();

        // 获取后一个规格的信息
        $nextSpec = array_pop($specs);
        $nextSpecAttrName = $nextSpec['attr_name'];
        $nextValueArray = $nextSpec['value_name'];

        if (!is_array($nextValueArray)) {
            $nextValueArray = array($nextValueArray);
        }
        // 对后一个规格的`value_name`数组进行遍历
        foreach ($nextValueArray as $k => $nextValue) {
            // 组成一个规格-属性数组
            $nextValueMergedSpec = array_merge(array('attr_name' => $nextSpecAttrName),
                                                array('value_name' => $nextValue)
                                            );
            // 对第一个规格的`value_name`数组进行遍历
            foreach ($valueArray as $key => $value) {
                // 组成一个规格-属性数组
                $valueMergedSpec = array_merge(array('attr_name' => $attr),
                                    array('value_name' => $value)
                                );
                $mergeArray[] = array_merge(array($nextValueMergedSpec),
                                    is_array($value) ? $value : array($valueMergedSpec)
                                );
            }
        }
        $valueArray = $mergeArray;
    }

    return $valueArray;
}
```

### 3.代码分解

(1)方法命名和参数确定

```php
/**
 * 商品规格排列组合
 * @param array  $specs
 * @return array $valueArray
 */
function goodsSpecPermutation($specs){}
```

参数形式：

```json
[
    {
        "attr_name": "颜色",
        "value_name": [
            "黄色",
            "绿色",
            "黑色"
        ]
    },
    {
         "attr_name": "尺码",
        "value_name": [
            "S",
            "M",
            "L",
            "XL"
        ]
    },
        {
        "attr_name": "赠品",
        "value_name": [
            "苹果",
            "香蕉",
            "梨子"
        ]
    }
]
```

(2)获取第一个规格的信息

```php
$spec = array_pop($specs);         // {"attr_name": "颜色","value_name": ["黄色", "绿色", "黑色", "黑色"]}
$attrName = $spec['attr_name'];    // "颜色"
$valueArray = $spec['value_name']; // ["黄色","绿色","黑色"]
```

> 执行出栈操作后，当前`$specs`的值为:

```json
    {
         "attr_name": "尺码",
        "value_name": [
            "S",
            "M",
            "L",
            "XL"
        ]
    },
        {
        "attr_name": "赠品",
        "value_name": [
            "苹果",
            "香蕉",
            "梨子"
        ]
    }
```

(3)获取下一个规格的信息

```php
$nextSpec = array_pop($specs); // {"attr_name": "尺码","value_name": ["S", "M", "L", "XL"]}
$nextSpecAttrName = $nextSpec['attr_name']; // "尺码"
$nextValueArray = $nextSpec['value_name'];  // ["S","M","L","XL"]
```

(4)遍历`$nextValueArray`，获取该规格下的一个属性值数组

```php
foreach ($nextValueArray as $k => $nextValue) { // $nextValue="S"
    $nextValueMergedSpec = array_merge(array('attr_name' => $nextSpecAttrName),
                                array('value_name' => $nextValue)
                            );
/* $nextValueMergedSpec = [
    "attr_name" => "尺码",
    "value_name" => "S"
]
*/
}
```