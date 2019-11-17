---
title: 【Python】循环结构
date: 2019-05-28 22:32:56
tags:
  - python
category:
  - 【Python相关】
---

掌握python循环语句的语法糖，学习python的for循环语句和其他语言的for循环的不同，以及python中平方`n**2`、立方`n**3`的表达方式，以及range()的特性，返回的并不是具体的列表

<!--more-->

### while循环

当条件成立时，循环体的内容可以一直执行，但是避免死循环，需要有一个跳出循环的条件才行。

### for循环

遍历任何序列（列表和字符串）中的每一个元素

```python
>>> a = ["China", 'is', 'powerful']
>>> for x in a:
...     print(x)
...
China
is
powerful
```

## range() 函数

生成一个等差数列(并不是列表)。

range(4) => `range(0, 4)`

list(range(4)) => `[0,1,2,3]`

list(range(1, 4)) => `[1,2,3]`

- `list()`: 返回一个序列（列表或字符串）中的每一个元素。
- range()返回的是一种可迭代对象，而不是具体的列表

```python
>>> a = range(10)
>>> a
range(0, 10)
>>> b = list(a)
>>> b
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
````

### continue语句

```python
#!/usr/bin/env python3
while True:
    n = int(input("Please enter an Integer: "))
    if n < 0:
        continue;
    elif n == 0:
        break;
    print("Square is ", n**2)
```

![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g3im99pphoj307w05gjrw.jpg)

- n的平方：`n**2`
- n的立方：`n**3`
