---
title:		Python Free Variables and Closures
subtitle:	Python Free Variables and Closures
summary:	Python Free Variables and Closures
date:		2020-05-25
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

# 由lambda引发的自由变量与闭包讨论

在上次学完lambda之后，我偶然在[一篇博客](https://www.cnblogs.com/xiangnan/p/3900285.html)中看到了如下的示例：

```python
#---CASE 1

fs = map(lambda i:(lambda j: i*j),range(6))
print([f(2) for f in fs])

#---CASE 2

fs = [lambda j:i*j for i in range(6)]
print([f(2) for f in fs])

#---CASE 3

fs = []
for i in range(6):
    fs.append(lambda j:i*j)
    if i==3:
        break
print([f(2) for f in fs])

#---CASE 4

fs = [(lambda i:lambda j:i*j)(i) for i in range(6)]
print([f(2) for f in fs])

[0, 2, 4, 6, 8, 10]
[10, 10, 10, 10, 10, 10]
[6, 6, 6, 6]
[0, 2, 4, 6, 8, 10]
```

这个例子还是蛮有意思的，在上边的四种情况中，只有CASE1和CASE4能够得到我们想要的结果。这是为什么呢？

其实如果从上次对于lambda的讨论直接入手来看，我们大概能发现一些规律，CASE1和CASE4有一个共同的特点，就是最外层函数中定义的变量作为了第二层（第三层）的**参数**，但是CASE2与CASE3并不具备这个特征，我们来看下深层的原因是什么。

Python是一种解释型语言，在Python解释器开始执行后，在内存中开辟了一个空间，如果遇到变量，那就把变量名和值的对应关系记录下来；但是在遇到函数时，解释器把函数读入内存之后，函数的内部逻辑，解释器此时并不知情，我们拿CASE2举例：

变量**i**对于匿名函数lambda来说，就是我们所说的自由变量（先不给准确定义，自由变量在这里提到只是为了方便理解和简化书写），我们把这个lambda匿名函数定义为f​. [原博客](https://www.cnblogs.com/xiangnan/p/3900285.html)的分析十分到位，这里我们直接拿来用：

> **函数f​**在定义时，解释器并没有把变量i和我们看到的对应的for循环中的值捆绑在一起，而只是明确了：在f​中，存在一个名为i的自由变量；
>
> 在函数f​被调用时，解释器会明确：
>
> 1. 空间上：f​要到**被定义时**的外层作用域查找自由变量i对应的对象，假设这个namespace为X.
> 2. 时间上: 是在你**当前运行时**, X 里面的 i 对应的对象

那么很显然，CASE2中的i所用的值都是5，因为调用的时候i值已经更新到了for循环中的最后一个。

但是对于CASE1和CASE4来讲，自由变量i不再是循环变量i，而是循环变量i所指对象在栈上的拷贝，由于每一次i所指对象都不相同，所以函数f​的自由变量自然而然地指向了不同对象；那么为什么会发生这种情况呢？在讲匿名函数lambda的那一篇中我们已经提到，这其实相当于在函数参数中给了一个默认参数(x=i)，由于默认参数的作用机制，**必须马上初始化默认参数**，自由变量被指向了不同的值。

形象化的定义这个过程，就出现了Python的**延迟绑定（迟绑定）**机制。啥叫迟绑定呢？那就是你以为他该绑定了，结果没绑定，绑定的晚了呗，这个论述的对象就是我们前边一直在说的**自由变量(free variable)**。

## 自由变量

定义：

> If a name is bound in a block, it is a **local variable** of that block, unless declared as nonlocal. If a name is bound at the module level, it is a **global variable**. (The variables of the module code block are local and global.) If a variable is used in a code block but not defined there, it is a **free variable**.
>
> --Python doc.

用不太标准的语言去解释，就是在一个代码块内部被调用，然而却在不在本代码块内定义，在外部代码块里定义的变量，就叫自由变量。

有了自由变量的概念，下边我们该看闭包了。

## 闭包

闭包定义：(来自Wikipedia)

> **闭包**:英语Closure，又称词法闭包（Lexical Closure）或函数闭包（function closures），是引用了自由变量的函数。这个被引用的自由变量将和这个函数一同存在，即使离开了创造它的环境也不例外。所以有另一种说法认为闭包 **是由函数和与其相关的引用环境组合而成的实体**。闭包在运行时可以有多个实例，不同的引用环境和相同的函数组合可以产生不同的实例。

从知乎上copy个小例子：

```python
def f1():
    x = 88
    def f2():
        print(x)
    return f2

action = f1()
action()

88
```

f1是外部函数，f2引用了f1中定义的自由变量x，我们通过变量action获取了返回的f2，这个时候，f1已经退出了，但是！f2还是记住了f1嵌套作用域当中的变量名x。这种现象就是**闭包**，在闭包现象中，我们可以发现在外部函数运行结束后，**自由变量并没有被垃圾回收机制回收**。

如果我们总结一下上边的例子，那就是：

**闭包中有一个能记住嵌套作用域变量的值的函数，即使作用域已经不再存在。换而言之，闭包的特性允许定义域非全局作用域的内部函数在定义时记得他们的外层作用域的样子。**

### 闭包特点

1. 避免使用全局变量，从而提供对某些数据的隐藏
2. 相当于对代码进行了封装，提高了可复用性和面向对象编程的优雅程度
3. 有利于并行计算

### 关于闭包的深层次研讨

在对于闭包有了初步的认识后，我们来更加深入的看下闭包的特性。

```python
def out_f():
    free_list = []
    def in_f(name):
        free_list.append(len(free_list) + 1)
        print('%s free_list = %s' %(name, free_list))
    return in_f

test_0 = out_f()
test_0('test_0')
test_0('test_0')
test_0('test_0')
test_1 = out_f()
test_1('test_1')
test_0('test_0')
test_1('test_1')
# 运行结果

test_0 free_list = [1]
test_0 free_list = [1, 2]
test_0 free_list = [1, 2, 3]
test_1 free_list = [1]
test_0 free_list = [1, 2, 3, 4]
test_1 free_list = [1, 2]
```

仔细观察上边的例子，我们可以得到如下结论：

1. 一个闭包中引用的嵌套作用域中的自由变量仅与此闭包有关联，与其他闭包无关，换言之，闭包的每个实例互不干扰。
2. 对于当前的闭包实例，对其中蕴含的自由变量的修改会被传递到下一次对当前闭包实例的调用。

> 对于闭包实例的调用，我们可以理解成让闭包执行一下，记一下嵌套作用域的环境，把该做的准备做好，就好比是在背文章之前总得先看一遍，之后就用就可以了

这两条性质其实不难理解，虽然函数作用域消失，但是根据之前提到的闭包特性我们知道，通过返回值，函数保留了嵌套作用域中的变量，所以自然而然的，对自由变量的修改会一直传递下去，除非我们设定了一个新的变量，那调用的函数其实就指向了另一片内存空间，如果我们第二次调用函数时返回的仍然是test_0,那么就相当于清零，传递从头开始。

而之前我们提到的CASE2与CASE3，则是典型的**闭包陷阱**。

看完了闭包的分析，我们就拿应用练练手，正好自己要学习Python的装饰器，那我们就开始装饰器的研讨。