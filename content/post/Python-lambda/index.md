---
title:		Python lambda function
subtitle:	Python lambda function
summary:	Python lambda function
date:		2020-05-24
lastmod:	2023-08-31
author:		shaopu
draft: 		false
image:		  
  focal_point: ''
  placement: 2
  preview_only: true

tags:
    - Python

categories:
    - Programming Language
---

{{< toc >}}

## 匿名函数lambda用法总结

既然lambda被称为匿名函数，那么我们只需要把握一点，即我们可以用
函数的观点来理解lambda的书写和调用方式,它返回的是一个函数对象。
### lambda的一般使用方法
本小节内容转载自<https://juejin.im/post/5d00b7efe51d4555fd20a31c>
#### 无参匿名函数
```python
# 可以将lambda直接传递给一个变量，像调用一般函数一样使用

B = lambda :True
print(B())
# 等价于

def BF():
 return True
print(BF())
# 示例结果

True
True
```
#### 有参匿名函数
支持单参数多参数，同时也可以带默认值，
这里举一个多参数的例子：

```python
two_sum = lambda x, y: x + y
# 等同于：

def two_sum(x, y): return x + y
print(two_sum(1,2))
# 示例结果

3
```
我们也可以直接从后边传参：
```python
two_sum = (lambda x, y: x + y)(3, 4)
print(two_sum)
# 示例结果

7
```

### lambda的高阶应用方法

#### 与list的结合（字典同理）
```python
list_a = [lambda a: a**3, lambda b: b**3]
# 是一个函数对象

print(list_a[0])
<function <lambda> at 0x0259B8B0>

g = list_a[0]
# 把其中的值拿出来

print(g(2))
8
```
#### 三元表达式结合求最小值
```python
lower = lambda x,y: x if x<y else y
print(lower(10,100))
```
#### 与map/reduce/filter/sorted函数的结合
之所以把这几个函数拿出来，是因为lambda的使用在调用这几个函数时格外常见，
因为可以大大缩减代码量

本小节内容参考CSDN用户-浅浅爱默默

##### filter()
filter函数的作用是过滤序列，把满足条件的元素留下来，返回一个迭代器对象（意味着如果想要列表需要手动转换），

语法：
```python
filter(function, iterable)
```
```python
result=[('G1','2','song'),('D2','2',''),('T3','','shaopu'),('K4','','3')]
print(list(filter(lambda i: i[0][0] in "GD",result)))
print(list(filter(lambda i: i[0][0] not in "GD",result)))

[('G1', '2', 'song'), ('D2', '2', '')]
[('T3', '', 'shaopu'), ('K4', '', '3')]
```
##### map()
map函数的作用是根据提供的函数对指定序列做映射,

语法：
```python
map(function, iterable, ...)
```
```python
#遍历map，根据不同的对象输出不同的结果，map(操作，操作对象)

a=[5,6,7,8,9,10,11,12]
def plus(i):             
    return i+10

print(map(plus,a))        
#计算列表a中各个元素，返回一个迭代器

print(list(map(plus,a)))  
#以列表形式返回映射后的结果

print(map(lambda i:i+10,a))      
#使用lambda匿名函数，返回一个迭代器

print(list(map(lambda i:i+10,a))) 
#以列表形式返回映射后的结果
 
# 运行结果：

<map object at 0x000001D694A89550>
[15, 16, 17, 18, 19, 20, 21, 22]
<map object at 0x000001D694A89128>
[15, 16, 17, 18, 19, 20, 21, 22]
# 提供了两个列表，对相同位置的列表数据进行相加

print(list(map(lambda x, y: x + y, [1, 3, 5, 7, 9], [2, 4, 6, 8, 10])))
# 运行结果：

[3, 7, 11, 15, 19]
```
##### reduce()
reduce函数的作用是对序列中的元素进行递归操作。reduce() 函数会对参数序列中元素进行累积。
函数将一个数据集合（链表，元组等）中的所有数据进行下列操作：用传给 reduce 中的函数 function（有两个参数）先对集合中的第 1、2 个元素进行操作，
得到的结果再与第三个数据用 function 函数运算，最后得到一个结果。

> 在py3中需要从functools导入

语法：
```python
reduce(function, iterable[, initializer])
```
```python
#reduce()递归，两个参数

from functools import reduce
print(reduce(lambda i,j:i*j,range(1,10)))  #1*2*3*4*5*6*7*8*9(含左不含右）

print(reduce(lambda i,j:i+j,range(1,10)))  #1+2+3+4+5+6+7+8+9
```

##### sorted()

语法：
```python
sorted(iterable[, key[, reverse]])
```
这个例子看下图像处理部分的调用就可以了:
```python
(cnts, boundingBoxes) = zip(*sorted(zip(cnts, boundingBoxes), key=lambda b: b[1][i], reverse=reverse))
```

#### for循环与lambda

本小节内容参考<https://www.cnblogs.com/liuq/p/6073855.html>

- **f = [lambda x: x*i for i in range(4)]**

这种表达形式等价于：
```python
def func():
    fs = []
    for i in range(4):
        def lam(x):
            return x*i
        fs.append(lam)
    return fs
```
当调用func()时，每循环一次，将lam函数的地址存到fs中。
因为在每次循环中lam函数都未绑定i的值，所以直到循环结束，
i的值为3，并将lam中所用到的i值定为3 ，因此真正调用（例如f[0](3)）的时候i值保持不变（为3）

```python
>>> f = [lambda x:x*i for i in range(4)]
>>> f[0](1)
3    # 1*3

>>> f[1](1)
3    # 1*3

>>> f[2](1)
3    # 1*3

>>> f[3](1)
3    # 1*3

>>> f[0](3)
9    # 3*3

>>> f[1](3)
9    # 3*3

>>> f[2](3)
9    # 3*3

>>> f[3](3)
9    # 3*3
```
>如果把x换成i，那就与传入的值无关了，始终是3*3
>
>```python
>f = [lambda :i*i for i in range(4)]
># f = [lambda i:i*i for i in range(4)]
>```

- **f1 = [lambda i=i: i*i for i in range(4)]**

这种表达形式等价于：
```python
def func():
    fs = []
    for i in range(4)
        def lam(x=i):   # 即 i=i
            
            return x*x    # 即 i*i
        
        fs.append(lam)
    return fs
```
当调用 func() 时，每循环一次，将lam函数的地址存到
fs中。但是在每次循环中lam函数都将i值绑定到了x上，**函数在定义时因为默认参数的存在，就给形参赋上了初值，**
所以直到循环结束，不同地址的lam函数的x值为都不一样,
因此真正调用（例如 f[0]）的时候x值都为当时被绑定的值。

```python
>>> f1 = [lambda i=i: i*i for i in range(4)]
>>> f1[0]()
0
>>> f1[1]()
1
>>> f1[2]()
4
>>> f1[3]()
9
```
这也意味着如果我们给x赋了新值，那么结果也会随之改变。

- **f2 = [lambda x=i: i*i for i in range(4)]**

这种表达形式等价于：

```python
def func():
    fs = []
    for i in range(4)
        def lam(x=i):
            return i*i
        fs.append(lam)
    return fs
```
虽然lam函数将i的值绑定到了x上，但函数体中并未使用x，所以直到循环结束，i的值变为3，才会在调用时使用。其实同第一种情况是一样的。

```python
>>> f2 = [lambda x=i: i*i for i in range(4)]
>>> f2[0]()
9
>>> f2[1]()
9
>>> f2[2]()
9
>>> f2[3]()
9
>>> f2[0](7)
9
>>> f2[1](7)
9
>>> f2[2](7)
9
```

在之后的讨论中，我们会再看几个稍微复杂的lambda嵌套的例子，并由此引出一个深坑--闭包。