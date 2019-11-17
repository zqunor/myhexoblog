---
title: 【Python】变量和数据类型
date: 2019-05-13 23:41:25
tags:
  - python
category:
  - 【Python相关】
---

之前看python做2048游戏的项目，直接上手，有一些复杂的地方不是太理解，所以开始学点基础语法再动手实践。学习素材是实验楼的python3教程，通过开发环境直接练习python的基础语法，包括关键字、循环、输入输出等基础的语法糖。

<!-- more -->

### 关键字

执行`$ python3`，进行到python3的终端`>>>`，输入` help()`，进入到`help>`，然后`keywords`查看python的关键字,有：
```python
False               def                 if                  raise
None                del                 import              return
True                elif                in                  try
and                 else                is                  while
as                  except              lambda              with
assert              finally             nonlocal            yield
break               for                 not
class               from                or
continue            global              pass
```

### 从键盘读取输入

- input(): 输出并接收一个用户输入值
- int(): 类型约束为整型，同样有float()
- if-else：条件判断
- print(): 打印输出

```python
#!/usr/bin/env python3
number = int(input("Enter an integer: "))
if number <= 100:
    print("Your number is less than or equal to 100")
else:
    print("Your number is greater than 100")
```

### while条件判断
- "Year {} Rs. {:.2f}".format(year, value)  字符串格式化
> format()中的参数依次填入到花括号中
- :.2f 格式约束为两位小数的浮点数

```python
#!/usr/bin/env python3
amount = float(input("Enter amount：")) # 输入数额
inrate = float(input("Enter Interest rate: ")) # 输入利率
period = int(input("Enter period: ")) # 输入期限
value = 0
year = 1
while year <= period:
    value = amount + (inrate * amount)
    print("Year {} Rs. {:.2f}".format(year, value))
    amount = value
    year = year + 1
```

### 实例

1、求N个数字的平均值

```python
#!/usr/bin/env python3
N = int(input("请输入N的值："))
sum = 0
count = 0
print("请输入N个数的值：")
while count < N:
    number = float(input())
    sum += number
    count += 1
avg = sum / N
print("N = {}, Sum = {}, Average = {:.2f}".format(N, sum, avg))
```

- 不支持`count++`

2、温度转换

摄氏度C = (华氏度F - 32) / 1.8
```python
#!/usr/bin/env python3
f = 0
print("Fahrenheit Celsius")
while f <= 250:
    c = (f - 32) / 1.8
    print("{:5d} {:7.2f}".format(f, c))
    f += 25
```

### 单行定义多个变量或赋值 -- 元组(tuple)

在一行内将多个值赋值给多个变量（从左到右，依次一个个赋值）

```python
>>> a, b = 45, 54
>>> a
45
>>> b
54
```

元组：用逗号创建元租。

在赋值语句右边创建一个元组，称之为元组封装(tuple packing),赋值语句的左边称作元组拆封(tuple unpacking)

```python
>>> data = ("tree", "flower", "grass")
>>> thing1, thing2, thing3 = data
>>> thing1
'tree'
>>> thing2
'flower'
>>> thing3
'grass'
```

学习材料：[实验楼 - Python3 简明教程
](https://www.shiyanlou.com/courses/596)