---
title: 【Python小试牛刀】循环
date: 2019-05-30 22:38:49
tags:
  - python
category:
  - 【Python相关】
---

使用python语法，完成简单的算法，包括斐波那契数列、幂级数、乘法表、星号组成的基本形状，以及棍子游戏。熟练掌握python的输入输出，以及循环结构在解决问题中的具体应用。

<!--more-->

### 1、斐波那契数列

斐波那契数列，数列前两项为1，之后每一项都是前两项之和。

```python
#!/usr/bin/env python3
a, b = 0, 1
while b < 100:
    print(b)
    a, b = b, a + b
```

> 默认print输出结果后会自动换行，如果不希望换行，只做间隔的话，就通过另一个参数end来替换这个换行符
```python
print(a, end=' ')
```

### 2、幂级数。

写一个程序计算幂级数：e^x = 1 + x + x^2 / 2! + x^3 / 3! + ... + x^n / n! （0 < x < 1）。

```python
#!/usr/bin/python3
x = float(input("Enter the value of x:"))

n = term = 1
result = 1.0
while n <= 100:
    term *= x/n
    result += term
    n += 1
    if term < 0.0001:
        break
print("No of Times={} and Sum = {}".format(n, result))
```

### 3、乘法表

打印10以内的乘法表。

```python
#!/usr/bin/env python3
i = 1
print('-' * 60)
while i < 11
    n = 1
    while n <= 10:
        print("{:d}*{:d}={:d}".format(n, i, i * n), end=" ")
        n += 1
    print()
    i += 1
print('-' * 60)
```

- print('-' * 60)：一个字符串重复60次输出

### 4、打印星号

打印各种形状的星号

- 向上的直角三角

```python
#!/usr/bin/env python3
n = int(input('Enter the number of rows:'))
i = 1
while i <= n:
    print('*' * i)
    i += 1
```

- 向下的直角三角

```python
#!/usr/bin/env python3
n = int(input('Enter the number of rows:'))
i = n
while i > 0:
    x = '*' * i
    y = " " * (n - i)
    print(y + x)
    i -= 1
```

- 菱形

```python
#!/usr/bin/env python3
n = int(input('Enter the number of rows:'))
i = 1
while i < n:
    x = '*' * (2 * i - 1)
    y = " " * (n - i)
    print(y + x)
    i += 1
while i > 0:
    x = " " * (n - i)
    y = '*' * (2 * i - 1)
    print(x + y)
    i -= 1
```

### 5、棍子游戏

有21根棍子，用户选1-4根棍子，然后电脑选1-4根棍子。谁选到最后一根棍子谁就输。（用户和电脑一轮选的棍子总数只能是5）

```python
#!/usr/bin/env python3
sticks = 21

while True:
    print("Sticks left: ", sticks)
    sticks_token = int(input("Take sticks(1-4):"))
    if sticks == 1:
        print("Failed!")
        break
    if sticks_token >= 5 or sticks_token <= 0:
        print("Choose wrong number! continue:")
        continue
    print("computer took:", 5-sticks_token, "\n")
    sticks -= 5
```

注：结果是必输无疑，哈哈!