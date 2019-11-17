---
title: 【Python】运算符
date: 2019-05-14 22:44:54
tags:
  - python
category:
  - 【Python相关】
---

python是强类型语言，某些场合下需要进行类型转换。关系运算符的结果是`true`或`false`。这里介绍一下基本的运算符，和几个使用实例，了解并掌握python中`range()`函数和`math类库`的引入和成员方法的调用

<!-- more -->

### 知识点

#### 整数运算符

- 整除：//
- divmod(num1, num2): 返回两个元素的元组， 第一个是num1和num2相整除得到的值，第二个是num1和num2求余得到的值
- *运算符：拆封元组

![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g318pjm253j30ft02b0t2.jpg)

#### 关系运算符

关系运算符 | 意义
-- | --
< | 小于
<= | 不大于
> | 大于
>= | 不小于
== | 等于
!= | 不等于

#### 逻辑运算符
- 逻辑运算符的优先级低于关系运算符
- 逻辑运算符优先级：`not` > `and` > `or`
- 逻辑运算符 and 和 or 也称作短路运算符：它们的参数从左向右解析，一旦结果可以确定就停止。短路运算符的返回值通常是能够最先确定结果的那个操作数。

```python
>>> 5 and 4
4
>>> 0 and 4
0
>>> False or 3 or 0
3
>>> 2 > 1 and not 3 > 5 or 4
True
```

#### 类型转换

类型转换函数 | 转换路径
-- | --
float(string) | 字符串 -> 浮点值
int(string) | 字符串 -> 整数值
str(integer) | 整数值 -> 字符串
str(float) | 浮点值 -> 字符串

### 实例

1、计算数列 `1/x + 1/(x+1) + 1/(x+2) + ... + 1/n`， 设 x = 1, n = 10。

```python
#!/usr/bin/env python3
sum = 0
for i in range(1, 11):
    sum += 1.0 / i
    print("{:2d} {:6.4f}".format(i, sum))
```

- `range(num1, numb2)`: 在num1-num2的范围内，不包括num2

2、输入三个数，并求解三个数组成的二次方程。（有解条件：`b*b-4*a*c>=0`）

```python
#!/usr/bin/env python3
import math
a = int(input("Enter value of a:"))
b = int(input("Enter value of b:"))
c = int(input("Enter value of c:"))
d = b * b - 4 * a * c
if d < 0:
    print("ROOTS are imaginary")
else:
    root1 = (-b + math.sqrt(d)) / (2 * a)
    root2 = (-b - math.sqrt(d)) / (2 * a)
    print("ROOT 1 = ", root1)
    print("ROOT 2 = ", root2) # 等价于print("ROOT 2 = {}"。format(root1=2))
```

- 引入数学函数库。`import math`
- 调用绝对函数。`math.sqrt()`
- 数学常量：`math.pi`