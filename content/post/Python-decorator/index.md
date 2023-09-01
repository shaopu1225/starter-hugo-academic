---
title:		Python Decorator
subtitle:	Python Decorator
summary:	Python Decorator
date:		2020-05-27
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

# 装饰器

记得上篇文章中**闭包**的基本构型吗？

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

我们对它稍微做点改变：

```python
def f1(x):
    def f2():
        print(x)
    return f2

action1 = f1(1)
action2 = f1(2)
action1()
action2()

1
2
```

我们把定义于外层函数内部的变量放到了参数的位置上，不过没关系，根据我们已知的闭包特性：函数记住了他的外层作用域，这么做完全可以。那么问题来了，如果我们把函数作为外部函数的参数传进去会发生什么呢？这就是我们常说的装饰器。

```python
def outer(some_func):
	def inner():
    	print "before some_func"
       	ret = some_func() # 1
        return ret + 1
	return inner
def foo():
	return 1
decorated = outer(foo) # 2

decorated()

>>> before some_func
>>> 2
```

本示例来自与<http://simeonfranklin.com/blog/2012/jul/1/python-decorators-in-12-steps/#section_10>

在这里，变量`decorated`就是`foo`的一个**装饰器**，装饰器，顾名思义，就是在原先的基础上增加了某些功能，那么有人会问，为啥增加功能非得这么个增加法？我直接把函数调用写在另一个函数里不就完事了？

当然，这么做没有问题，但是闭包的优点我们已经写过了不是吗？简而言之，我们还是想保留原先的代码结构。

于是，在上边`decorated`的基础上，我们可能想直接用装饰后的版本替换原先的`foo`函数，得到一个新的`foo`版本，我们只需要把#2换成：

``````python
foo=outer(foo)
foo()
``````

在很多情况下，之前的foo并不出现在inner()中直接调用，而是作为了inner的返回值：

``````python
def outer(some_func):
    def inner():
        print "before some_func"
        return some_func()
    return inner
def foo():
    return 1
decorated = outer(foo) # 2

decorated()
``````

## 提到装饰器，不能不提语法糖@

这个语法糖其实就是为了方便操作和编码准备的，没啥难的，网上教程一搜一大堆。

把上边的例子改改就行了：

``````python
def outer(some_func):
    def inner():
        print "before some_func"
        return some_func()
    return inner

@outer
def foo():
    return 1
# foo = outer(foo) # 2

foo() # decorated()
``````

把@放在我们需要的原函数定义的地方，就可以省略最后一步中的赋值操作。

## *args、**kwargs

首先我们得搞清楚这俩玩意儿是干嘛的。

简单地讲，这两个东西均隶属于可变参数的范畴，也就是说可以不定量传递，分别叫做**包裹位置参数和包裹关键字参数**，在一般的Python函数的定义中，我们要按照如下形式定义参数：

`位置参数、默认参数、包裹位置参数、包裹关键字参数`

其中，位置参数就是最一般的根据所在位置决定的参数，在函数的调用中，具体那个参数属于哪一类，这是要取决于前后的参数和自己所在的位置的，这也是我们最常见的参数类型，如果再细分，位置参数还可以分为位置参数与关键字参数，其中，关键字参数是以"键-值"对的形式给出的，而这两种参数类型，也分别恰好对应了包裹位置参数与包裹关键字参数。

对于包裹位置参数，其实就是一个元组，如果我们最后输出，会发现其实Python把他们按照**元组**的形式打包了，那么同理，包裹关键字型参数就是按照**字典**形式打包的。

通过他们，我们可以给原函数传递任意数量和形式的参数，举个例子：

``````python
def logger(func):
	def inner(*args, **kwargs): # 1
        
		print "Arguments were: %s, %s" % (args, kwargs)
		return func(*args, **kwargs) # 2
    
	return inner

@logger
def foo(x, y=2, city='Nanjing'):
	return x * y
# foo = logger(foo)

k = foo(1, city='Beijing')
print(k)

>>> Arguments were: (1,), {'city':'Beijing'}
>>> 2
``````

这种参数表示法的好处就在于我们可以将所有的，不论什么形式的`foo`的参数都可以完整的传递到`func()`中去. 这个函数其实很好理解，在之前的叙述中，我们提到装饰器函数记住了自己的外层作用域，其实也就是与外部的自由变量绑定在了一起，我们通过`@logger`获取了`logger`的返回值`inner`函数，并与变量名`foo`关联. 调用`foo(1,city='Beijing')`时，也即执行`inner(1,city='Beijing')`，因为我们调用函数时**包裹位置参数**中只出现了`x`，所以参数列表中元组内只有`x`的值。但因为`foo(x,y=2,city='Nanjing')`中默认参数的存在，其返回值变成了2.

## 带参数的装饰器

现在提出一个问题，我想让装饰器也接受另外的参数，做一个层级调度任务，该怎么办？

我们回去看一眼现在的装饰器长啥样儿：

``````python
def outer(some_func):
    def inner():
        print "before some_func"
        return some_func()
    return inner

@outer
def foo():
    return 1
# decorated = outer(foo) # 2

decorated()
``````

我们当时为什么要设计装饰器来着？是为了做改进但是又不破坏原有的代码结构，那岂不是和我们现在要做的是同一件事了？所以答案很简单，接着嵌套就完事了。

从廖大那儿搞个例子：

``````python
def log(text):
    def decorator(func):
        def wrapper(*args, **kw):
            print('%s %s():' % (text, func.__name__))
            return func(*args, **kw)
        return wrapper
    return decorator

@log('execute')
def now():
    print('2020-5-27')
``````

因为内层的函数始终能记住外部的嵌套作用域，所以这就无限套娃了。(外层函数的返回值应当是其内部函数名，这是必要的)

我觉得廖大的解释很棒，这个3层嵌套的效果是这样：

``````python
now = log('execute')(now)
``````

同理，首先执行`log('execute')`，返回的是`decorator`函数，再调用返回的函数，参数是`now`函数，返回值是`wrapper`函数，执行`now = log('execute')(now)`后，将函数`now`作为变量传入`log('execute')`，即`decorator`装饰器中，然后`now`方法在`decorator`中的`wrapper`函数实现，并包装新的功能. 所以`now`的新值是经过``decorator`装饰的`wrapper`方法. 

## 多装饰器

关于多装饰器，网上例子不少，但是我还是想写下，加深一下印象。

多装饰器的执行方法：

``````python
@f1
@f2
@f3
def f()
	pass

f = a(b(c(f)))
``````

基本上是一个从下至上的执行顺序。但是问题往往再不经意间，咱们接着往下看：

以下内容参考自<https://segmentfault.com/a/1190000007837364>

``````python
def decorator_a(func):
    print 'Get in decorator_a'
    def inner_a(*args, **kwargs):
        print 'Get in inner_a'
        return func(*args, **kwargs)
    return inner_a

def decorator_b(func):
    print 'Get in decorator_b'
    def inner_b(*args, **kwargs):
        print 'Get in inner_b'
        return func(*args, **kwargs)
    return inner_b

@decorator_b
@decorator_a
def f(x):
    print 'Get in f'
    return x * 2

f(1)
``````

``````python
Get in decorator_a
Get in decorator_b
Get in inner_b
Get in inner_a
Get in f
``````

很显然，根据装饰器一般说的从下至上的执行顺序来讲，这是不对的。所以到底是哪里出了问题呢？

我们把握两点：

1. 函数与函数调用的区别
2. 装饰器函数在装饰函数定义好后立即执行

- 关于第一点：

  >  回想一下闭包时我们对调用闭包实例的阐述，是不是一个意思？
  >
  > > 形象化表述：对于闭包实例的调用，我们可以理解成让闭包执行一下，记一下嵌套作用域的环境，把该做的准备做好，就好比是在背文章之前总得先看一遍，之后就用就可以了

  为什么是先执行 `inner_b` 再执行 `inner_a` 呢？为了彻底看清上面的问题，得先分清两个概念:函数和函数调用。上面的例子中 `f` 称之为函数， `f(1)` 称之为函数调用，后者是对前者传入参数进行求值的结果。**在Python中函数也是一个对象，所以 `f` 是指代一个函数对象，它的值是函数本身， `f(1)` 是对函数的调用，它的值是调用的结果，这里的定义下 `f(1)` 的值2。**同样地，拿上面的 `decorator_a` 函数来说，它返回的是个函数对象 `inner_a` ，这个函数对象是它内部定义的。在 `inner_a` 里调用了函数 `func` ，将 `func` 的调用结果作为值返回。

- 关于第二点：

  其实这一点，我们把语法糖的表示还原回去看的更清楚：

  ``````python
  @decorator_a
  def f(x):
      print 'Get in f'
      return x * 2
  
  # 相当于
  
  def f(x):
      print 'Get in f'
      return x * 2
  
  f = decorator_a(f)
  ``````

  所以，当解释器执行这段代码时， `decorator_a` 已经调用了，它以函数 `f` 作为参数， 返回它内部生成的一个函数，所以此后 `f` 指代的是 `decorater_a` 里面返回的 `inner_a` （因为装饰器的写法：`f=decorator(f)`）。所以当以后调用 `f`时，实际上相当于调用 `inner_a` ,传给 `f`的参数会传给 `inner_a` , 在调用 `inner_a` 时会把接收到的参数传给 `inner_a` 里的 `func` 即 `f` ,最后返回的是 `f` 调用的值，所以在最外面看起来就像直接在调用 `f` 一样

  上边的表述有点转圈，循环的感觉，但其实没毛病，因为本身我们的参数就是从`f`传进去，最后又传回了`f`，那我们做了啥呢？当然是做了些装饰啦。

于是，在最初的例子中，在我们定义了装饰器之后：

``````python
@decorator_b
@decorator_a
def f(x):
    print 'Get in f'
    return x * 2
``````

根据原则1，实际上按照从下到上的顺序已经依次调用了 `decorator_a` 和 `decorator_b` ，这是会输出对应的 `Get in decorator_a` 和 `Get in decorator_b` 。

那么接下来，由于多装饰器的嵌套的执行方法，`decorator_a`装饰器先return 了`inner_a`, 而`decorator_b`后面又把`inner_a`装饰了，最终整个暴露在外面的是`inner_b`，这时候 `f` 已经相当于 `decorator_b` 里的 `inner_b` 。但因为 `f` 并没有被调用，所以 `inner_b` 并没有调用，依次类推 `inner_b` 内部的 `inner_a` 也没有调用，所以 `Get in inner_a` 和 `Get in inner_b` 也不会被输出。

然后最后一行当我们对 `f` 传入参数1进行调用时， `inner_b` 被调用了，它会先打印 `Get in inner_b` ，然后在 `inner_b` 内部调用了 `inner_a` 所以会再打印 `Get in inner_a`, 然后再 `inner_a` 内部调用的原来的 `f`, 并且将结果作为最终的返回。

## 函数属性出了点问题？

紧接着上边的，看看这个：

``````python
print(now.__name__)
'wrapper'
``````

函数的名字在用了装饰器之后变了？

装饰器就是存在这个问题，他会替换函数的**元信息**，

那为什么会替换元信息呢？

我们还是拿之前的2层嵌套为例说明：

``````python
def outer(some_func):
    def inner():
        print "before some_func"
        return some_func()
    return inner

@outer
def foo():
    return 1
# decorated = outer(foo) # 2

decorated()
``````

之前我们分析过函数的调用顺序，通过调用顺序我们知道，调用`f`其实就是在调用`inner()`.这句话本身就是元信息替换的原因。

为了解决这个问题，Python自然有自己的办法：

``````python
import functools

def log(text):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kw):
            print('%s %s():' % (text, func.__name__))
            return func(*args, **kw)
        return wrapper
    return decorator
``````

这么说来，wraps也是一个装饰器，它的作用就是把原函数的信息拷贝到装饰器的func函数中。

## 类装饰器

一般依靠类的`__call__`方法：

```python
class Foo(object):
    def __init__(self, func):
        self._func = func

    def __call__(self):
        print ('class decorator runing')
        self._func()
        print ('class decorator ending')

@Foo
def bar():
    print ('bar')

bar()
```

输出：

``````python
class decorator runing
bar
class decorator ending
``````

装饰器的基本内容就学习到这儿，关于Python的类可以直接参看官方文档：

<https://docs.python.org/zh-cn/3/tutorial/classes.html>