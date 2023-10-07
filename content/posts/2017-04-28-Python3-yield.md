---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2017-04-28T12:48:40+08:00'
description: Python中的yield
lastmod: '2017-06-02T04:01:32+08:00'
showToc: true
tags: [Python]
title: 深入理解 Python中的yield
---

## 深入理解 Python3 yield

> `yield`是python内置的关键字，它能产生一个生成器（Generator）。

### 0x01. Generators

只要函数中有`yield`就会变成一个`generator object`生成器对象，生成器对象可以迭代，但与`iterable`不同的是它只能迭代一次。先来看一个简单的例子

    >>> def foo():
    ...     yield 1
    ...     yield 2
    ...     yield 3
    ... 
    >>> g = foo()
    >>> g
    <generator object foo at 0x7ffb08326ca8>
    >>> for i in g:
    ...     print(i)
    ...     
    ... 
    1
    2
    3
    >>> for i in g:
    ...     print(i)
    ...     
    ... 
    >>> 

当你调用`foo`这个函数的时候，函数内部的代码并不立马执行 ，这个函数只返回一个生成器对象，如果有庞大的数据它不像`iterable`占用内存，会每次调用时才计算产生值。

其实`for`循环隐式的调用`__next__()`方法，直到遇到`StopIteration`停止。

    >>> def bar():
    ...     yield 'a'     # 第一次调用next()代码运行到这，产生'a'
    ...     yield 'b'     # 第二次调用next()代码运行到这，产生'b'
    ...     yield 'c'     # 第三次调用next()代码运行到这，产生'c'
    ... 
    >>> g = bar()
    >>> next(g)
    'a'
    >>> next(g)
    'b'
    >>> next(g)
    'c'
    >>> next(g)
    Traceback (most recent call last):
      File "<input>", line 1, in <module>
        next(g)
    StopIteration

也可以使用Python3内置的`next()`函数调用，直到产生`StopIteration`错误。

下面是一个[斐波那契数列](https://en.wikipedia.org/wiki/Fibonacci_number)的例子

    >>> def fib(n):
    ...     a, b = 0, 1
    ...     while n >= 0:
    ...         b, a = a, a+b
    ...         n -= 1
    ...         yield b
    ...     
    ... 
    >>> for i in fib(5):
    ...     print(i)
    ...     
    ... 
    0
    1
    1
    2
    3
    5

### 0x02. send Method /Coroutines

`yield`不仅可以通过`next()`取得产生的值，还可以通过`send()`接受值。

    >>> def simple_coroutine():
    ...     print('coroutine has been started!')
    ...     x = yield
    ...     print('coroutine received:', x)
    ...     
    ... 
    >>> cr = simple_coroutine()
    >>> cr
    <generator object simple_coroutine at 0x7ffb08337a40>
    >>> next(cr)    # 等于send(None)
    coroutine has been started!
    >>> cr.send('Hi')
    coroutine received: Hi
    Traceback (most recent call last):
      File "<input>", line 1, in <module>
        cr.send('Hi')
    StopIteration

首先必须调用`next()`或`send(None)`开始这个生成器，程序运行到`x = yield`暂停，这就是`yield`神奇的地方可以**中断和恢复**。因为`yield`后面没有值，默认为`None`，之后`send('Hi')`，生成器接受`'Hi'`并把它赋值给左边的`x`继续运行。由于没有下一个`yield`就抛出了`StopIteration`错误。

`send(value)`会传递value给`yield`，并且返回下一个`yield`右边的值。`next()`也会传递给`yield`值只是值为`None`，返回`yield`右边的值，这个和`send`一样。所以第一次调用`next()`等同于`send(None)`开始一个生成器。

*具体什么是协程，我记录在了[这里](http://www.fythonfang.com/blog/post/12)，欢迎参阅。*

### 0x03. throw / close Method

`throw(type)`方法会在`yield`暂停的地方直接抛出一个为`type`的异常，程序退出。如果被`try...except..`捕获了就继续执行返回下一个`yield`值，之后没有`yield`的话就会抛出一个`StopIteration`错误。

`close()`是一个优雅地退出`generator`的方法，本质是抛出一个`GeneratorExit`异常。

    >>> def echo(value=None):
    ...     print("Execution starts when 'next()' is called for the first time.")
    ...     try:
    ...         while True:
    ...             try:
    ...                 value = (yield value)
    ...             except Exception as e:
    ...                 value = e
    ...     finally:
    ...         print("Don't forget to clean up when 'close()' is called.")
    ...         
    ...     
    ... 
    >>> generator = echo(1)
    >>> print(generator)
    <generator object echo at 0x7ffb0822ec50>
    >>> print(generator.send(None))
    Execution starts when 'next()' is called for the first time.
    1
    >>> print(next(generator))
    None
    >>> print(generator.send(2))
    2
    >>> generator.throw(TypeError, "spam")
    TypeError('spam',)
    >>> generator.close()
    Don't forget to clean up when 'close()' is called.

这个是[官方文档](https://docs.python.org/3/reference/expressions.html#generator.throw)上的例子，首先`send(None)`开始这个生成器，输出右边的`value 1 `，调用`next()`后输出`None`是因为`next()`默认传递`None`给左边的`value`然后到下一个`yield value`所以是`None`。`send(2)`只是传给`value`的值为`2`其他同理，所以输出是`2`。`throw()`会抛出一个异常，而被`except`捕获并赋值给了`value`。`close()`关闭了`generator`退出当前块代码运行`finally`代码块。

### 0x04. yield from

`yield from`是从`Python 3.3`之后加入的。

    >>> def gen1():
    ...     for char in "Python":
    ...         yield char
    ...     for i in range(5):
    ...         yield i
    ...     
    ... 
    >>> def gen2():
    ...     yield from "Python"
    ...     yield from range(5)
    ... 
    >>> g1 = gen1()
    >>> g2 = gen2()
    >>> for i in g1:
    ...     print(i, end=',')
    ...     
    ... 
    P,y,t,h,o,n,0,1,2,3,4,
    >>> for i in g2:
    ...     print(i, end=',')
    ...     
    ... 
    P,y,t,h,o,n,0,1,2,3,4,

上面的`gen1`和`gen2`是等价的但代码看上去更加的整洁。`yield from`后接可迭代对象(iterable)。然后我们看一个后面接生成器的例子。

    >>> def accumulate():
    ...     tally = 0
    ...     while True:
    ...         next = yield
    ...         if next == None:
    ...             return tally
    ...         tally += next
    ...         
    ...     
    ... 
    >>> def gather_tallies_verbose(tallies):
    ...     acc = accumulate()
    ...     next(acc)
    ...     while True:
    ...         try:
    ...             value = yield
    ...             acc.send(value)
    ...         except StopIteration as tally:
    ...             tallies.append(tally.value)
    ...             acc = accumulate()
    ...             next(acc)
    ...         except Exception as e:
    ...             print(e.value)
    ...             acc.throw(e)
    ...             
    ...         
    ...     
    ... 
    >>> def gather_tallies(tallies):
    ...     while True:
    ...         tally = yield from accumulate()
    ...         tallies.append(tally)
    ...         
    ...     
    ... 
    >>> tallies = []
    >>> acc = gather_tallies(tallies)
    >>> next(acc)
    >>> for i in range(4):
    ...     acc.send(i)
    ...     
    ... 
    >>> tallies
    []
    >>> acc.send(None)
    >>> tallies
    [6]
    >>> for i in range(5):
    ...     acc.send(i)
    ...     
    ... 
    >>> acc.send(None)
    >>> tallies
    [6, 10]

这个例子原来是[官方文档](https://docs.python.org/3/whatsnew/3.3.html#pep-380)上的例子我改了下。`gather_tallies_verbose`和`gather_tallies`两个是等价的，但`gather_tallies`使用了`yield from`对比发现更加简洁易懂，magic。

`accumulate()`是一个生成器对象，`gather_tallies()`也是一个生成器。子生成器`accumulate`能接受`gather_tallies`的`send()`过来的值，子生成器接受到`None`后`return`的值会传递给调用者(`gather_tallies`)并赋值在左边也就是`tally`然后继续运行，子生成器`return`其实会抛出一个`StopIteration`错误而捕获到错误的`value`值也就是`tally`。

具体可查看如下规则[`PEP 380 - Syntax for Delegating to a Subgenerator`](https://www.python.org/dev/peps/pep-0380/)

> * Any values that the iterator yields are passed directly to the caller.
> * Any values sent to the delegating generator using `send()` are passed directly to the iterator. If the sent value is None, the iterator's `__next__` method is called. If the sent value is not None, the iterator's `send()` method is called. If the call raises `StopIteration`, the delegating generator is resumed. Any other exception is propagated to the delegating generator.
> * Exceptions other than `GeneratorExit` thrown into the delegating generator are passed to the `throw()` method of the iterator. If the call raises `StopIteration`, the delegating generator is resumed. Any other exception is propagated to the delegating generator.
> * If a `GeneratorExit` exception is thrown into the delegating generator, or the `close()` method of the delegating generator is called, then the `close()` method of the iterator is called if it has one. If this call results in an exception, it is propagated to the delegating generator. Otherwise, `GeneratorExit` is raised in the delegating generator.
> * The value of the yield from expression is the first argument to the `StopIteration` exception raised by the iterator when it terminates.
> * `return expr` in a generator causes `StopIteration(expr)` to be raised upon exit from the generator.

**参考**

[http://www.python-course.eu](http://www.python-course.eu/python3_generators.php)

[https://docs.python.org](https://docs.python.org/3/reference/expressions.html#generator.throw)

[https://docs.python.org](https://docs.python.org/3/whatsnew/3.3.html#pep-380)