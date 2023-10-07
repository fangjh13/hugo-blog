---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2017-04-10T18:12:51+08:00'
description: 装饰器介绍
lastmod: '2020-04-13T03:14:56+08:00'
showToc: true
tags: [Python]
title: Python装饰器(decorators)
---

# Python装饰器(decorators)

装饰器(decorators)是Python强大的功能之一，语法上就支持(@符号)使用起来更方便，不需要用OOP的设计模式实现。装饰器其实就是个返回函数的函数(类)，但可以有很多的玩法，下面将一一介绍。

## 函数(Functions)

讲装饰器之前，先回顾下一些函数的基础知识，装饰器就是这些简单功能的组合

### 函数接收函数作为参数

python中定义一个函数很简单如下

```python
>>> def foo():
...     pass
...
>>> foo
<function foo at 0x1054157a0>
>>> bar = foo
>>> bar
<function foo at 0x1054157a0>
```

定义了`foo`函数，而`bar`是对`foo`的引用，这很简单

因为python中一切皆对象，函数也是对象，一个函数也可以使用函数作为参数传入，和传其他对象一样（字符串、数字、列表 ...)

```python
>>> def foo():
...     print("hello world")
...
...
>>> def bar(f):
...     print(f"call {f.__name__}")
...     f()
...
...
>>> bar(foo)
call foo
hello world
```

`bar`函数就接收`foo`函数作为参数，内部执行`foo`函数。

### 函数内部定义函数

也可以在函数内部定义一个新的函数

```python
>>> def foo():
...     def bar():
...         print("inner func")
...     bar()
...
...
>>> foo()
inner func
>>> bar()
Traceback (most recent call last):
  File "<input>", line 1, in <module>
    bar()
NameError: name 'bar' is not defined
```

`foo`函数中定义了`bar`函数，定义内部函数和定义在外面的函数没有任何的区别，只是它的作用域只能在`foo`函数内部，外部是无法应用`bar`的


### 函数返回函数

更高级的函数甚至可以返回一个函数作为返回结果

```python
>>> def foo():
...     def bar():
...         return "hello world"
...     return bar
...
>>> foo
<function foo at 0x10c063440>
>>> foo()
<function foo.<locals>.bar at 0x10baea170>
>>> foo()()
'hello world'
```

注意我们这一次内部不再调用`bar()`而是`return bar`，说明`foo`函数返回一个内部函数的引用

可以看到调用`foo()`函数返回了内部定义的`bar`函数(`<function foo.<locals>.bar at 0x10baea170>`)但没有执行调用，再次调用则会被执行。

## 装饰器

有了上面的基础，我们就可以创建装饰器了

```python
def first_decorator(f):
    def wrapper():
        print(f"call function {f.__name__!r}")
        f()
        print("call finished")
    return wrapper

def task():
    print("do some task...")

task = first_decorator(task)

task()
```

`task`函数作为`first_decorator`的参数传入然后重新赋值给了`task`变量，而`first_decorator`内部返回`wrapper`函数引用，执行`task`后如下结果

```shell
call function 'task'
do some task...
call finished
```

以上就是一个最简单装饰器(`first_decorator`)的例子，所以装饰器的作用就是**在不修改原函数(`task`)的基础上给原函数增加一些功能，它能包装原函数改变原函数的一些功能**。在一些场景下节省了很多代码量而且简单直观，比如权限验证、日志、缓存等。

### 语法糖

上面写了`task = first_decorator(task)`来实现包装的效果，python提供了一个更加优雅的语法糖那就是`@`符号，可以改写成这样

```python
def first_decorator(f):
    def wrapper():
        print(f"call function {f.__name__!r}")
        f()
        print("call finished")
    return wrapper

@first_decorator                  # 语法糖
def task():
    print("do some task...")
```

这是不是更简单了，`@first_decorator`的作用和`task = first_decorator(task)`一样

但被装饰的函数如果有参数怎么办呢，我们使用`*args`和`**kwargs`解决，下面的装饰器是记录某函数调用时间的

```python
import time

def time_cost(f):
    def wrapper(*args, **kwargs):               # 接受任何类型的参数
        start = time.perf_counter()
        result = f(*args, **kwargs)             # 被装饰的函数调用
        print(f"{f.__name__} run cost {time.perf_counter()-start:.5f}s")
        return result
    return wrapper

@time_cost
def doze(t: int) -> int:
    """sleep a time"""
    time.sleep(t)
    return t

doze(2)      # doze run cost 2.00506s
```

> 上面定义`doze`函数时使用了python`type hints`的特性，请使用3.6以上版本

使用`@time_cost`语法糖装饰`doze`函数，`doze`的输入参数其实最后是传给了`wrapper(*args, **kwargs)`，之后才会被使用，而我们暂存了结果等内部处理完再返回结果，不会影响被装饰函数的返回结果。

### 元数据

`doze`执行起来功能好像是对的，但它的一些属性可能会有些问题

```python
>>> doze.__name__
'wrapper'
>>> doze.__doc__

```

看到这些元数据都是引用内部定义的`wrapper`函数，因为被装饰了之后返回的是`wrapper`函数的引用，我们需要修复它

```python
import time

def time_cost(f):
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = f(*args, **kwargs)
        print(f"{f.__name__} run cost {time.perf_counter()-start:.5f}s")
        return result
    wrapper.__name__ = f.__name__
    wrapper.__doc__ = f.__doc__
    wrapper.__module__ = f.__module__
    return wrapper

@time_cost
def doze(t: int) -> int:
    """sleep a time"""
    time.sleep(t)
    return t

print(doze.__name__)    # doze
print(doze.__doc__)     # sleep a time
```

在`time_cost`内部我们手动修改了这些属性把原函数的一些属性赋值到了`wrapper`函数，其实`functools`包提供了一个`wraps`装饰器专门用来干这个事情，所以可以写成下面这样子

```python
import time, functools

def time_cost(f):
    @functools.wraps(f)             # 修复元数据
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = f(*args, **kwargs)
        print(f"{f.__name__} run cost {time.perf_counter()-start:.5f}s")
        return result
    return wrapper
```

注意`functools.wraps`也是一个装饰器，`@functools.wraps`装饰了`wrapper`函数使得`wraper`上的元数据和`f`函数一致

### 传参

能不能在装饰的时候传入参数呢如`@decorator(k=v)`使装饰器更加灵活呢，答案是肯定的。

要想`@decoraotr(k=v)`可用，**`decorator(k=v)`整体就要返回一个函数引用**，此函数用来装饰目标对象（接收一个函数），模版如下

```python
def decorator(k):
    def dec_args(f):
        # 和之前定义的装饰器一样
        # ...
    return dec_args

@decorator(3)
def foo:
    #...
```

简单理解就是多增加一层嵌套为的是传入`k`，`decorator(k)`返回的函数引用就是用来装饰目标函数的，接受目标`f`(被装饰的目标函数)，此`dec_args(f)`和之前定义的不带参数的装饰器一样（如上面的`time_cost`,`first_decorator`），如下是一个可以指定执行次数的装饰器

```python hl_lines="1 9"
def repeat(n):
    def dec_args(f):
        @functools.wraps(f)
        def wrapper(*args, **kwargs):
            for i in range(n):
                result = f(*args, **kwargs)
            return result
        return wrapper
    return dec_args

@repeat(n=3)
def echo(s):
    print(s)

echo("hello world")
```

`repeat`装饰器接受参数`n`执行的次数，不用语法糖手写就是`echo = repeat(n=3)(echo)`，输出

```shell
hello world
hello world
hello world
```

如何实现即可以传参(`@repeat(n=3)`)又可以省略参数(`@repeat`)呢，这需要一点小trick，内部增加一个判断

```python
import functools

def repeat(_func=None, *, n=1):
    def dec_args(f):
        @functools.wraps(f)
        def wrapper(*args, **kwargs):
            for i in range(n):
                result = f(*args, **kwargs)
            return result
        return wrapper
    if _func is None:
        return dec_args            # 传参的情况
    else:
        return dec_args(_func)     # 没有参数

@repeat
def echo(s):
    print(s)

@repeat(n=3)
def echo3(s):
    print(s)

echo("hello world")
echo3("hello friend")
```

我们使用`*`符号确保传参必须使用键值对，使用`_func`变量判断有没有传参，分两种情况

1. 传参: `echo3 = repeat(n=3)(echo3)`所以`_func`是`None`
2. 没有参数: `echo = repeat(echo)`所以`_func`是函数`echo`

还有一种使用`functools.partial`的实现方法

```python
def repeat(_func=None, *, n=1):
    if _func is None:                          # 带参数
        return functools.partial(repeat, n=n)
    @functools.wraps(_func)
    def wrapper(*args, **kwargs):
        for i in range(n):
            result = _func(*args, **kwargs)
        return result
    return wrapper
```

### 装饰器级连

一个函数可以使用多个装饰器，我们把`repeat`和`time_cost`两个装饰器都作用在`echo`上

```python
@time_cost
@repeat(n=3)
def echo(s):
    time.sleep(0.5)
    print(s)
```

输出

```shell
hello world
hello world
hello world
echo run cost 1.50582s
```

上面两个装饰器就是等于`echo = time_cost(repeat(n=3)(echo))`，当然也可以用三个以此类推

### 装饰器与类

到现在为止我们都是用函数定义装饰器，使用在函数上，接下来介绍装饰器与类有关的操作

- 装饰的对象为类或者类方法
- 使用类定义装饰器

首先看装饰在类方法上，其实和装饰在函数上是一样的，本来定义在类中的函数叫类方法

```python
class Task:
    @time_cost
    def echo(self):
        print("hello world")

Task().echo()
```

上面作用在`echo`方法上显示方法耗时，python提供了一些内建的用于类相关的装饰器如`@classmethod`类方法、`@staticmethod`静态方法、`@property`属性

我们把装饰器用在类上面

```python
@time_cost
class Task:
    def echo(self):
        print("hello world")

t = Task()
print("-" * 10)
t.echo()
```

输出

```shell
Task run cost 0.00000s
----------
hello world
```

这个是作用在类实例化上，不会对类方法有什么作用，`time_cost`接收的是类`Task`，`Task = time_cost(Task)`

以上我们都用的是函数定义装饰器，装饰器也可以用类来定义主要使用类的`__call__`方法，先来看看`__call__`方法

```python
>>> class Counter:
...     def __init__(self):
...         self.n = 0
...
...     def __call__(self):
...         self.n += 1
...         print(f"{self} call {self.n} times")
...
...
>>> c = Counter()
>>> c()
<__main__.Counter object at 0x10c5b2b90> call 1 times
>>> c()
<__main__.Counter object at 0x10c5b2b90> call 2 times
>>> c()
<__main__.Counter object at 0x10c5b2b90> call 3 times
```

[`__call__`](https://docs.python.org/3/reference/datamodel.html#object.__call__)方法在实例自身调用时触发，这里记录每次调用的次数。用类定义装饰器就是定义`__call__`和`__init__`方法

```python
import functools

class Counter:
    def __init__(self, f):
        functools.update_wrapper(self, f)
        self.n = 0
        self.f = f            # 被装饰的对象

    def __call__(self, *args, **kwargs):
        self.n += 1
        result = self.f(*args, **kwargs)
        print(f"{self.f.__name__} call {self.n} times")
        return result

@Counter
def run_task():
    pass

run_task()
run_task()
```

输出

```shell
run_task call 1 times
run_task call 2 times
```

`Counter`就是用类定义的装饰器相当于`run_task = Counter(run_task)`，`__init__`方法在此时调用`self.f`保存函数引用，而`__call__`就是在被装饰的函数调用的时候触发，每次自增1。注意我们用`functools.update_wrapper`更新元属性而不是用`functools.wraps`装饰器其实`wraps`装饰器内部也是调用`update_wrapper`的，如果漏了这句被装饰对象(`run_task`)的类似`__name__`和`__doc__`等会丢失。

当然上面的计数装饰器也可以用普通函数来实现，内部需要有个变量保存调用次数，我们使用函数属性`wraper.n`

```python
def counter(f):
    @functools.wraps(f)
    def wrapper(*args, **kwargs):
        wrapper.n += 1
        result = f(*args, **kwargs)
        print(f"{f.__name__} call {wrapper.n} times")
        return result
    wrapper.n = 0
    return wrapper
```

## 应用举例


### 缓存

因为装饰器内部可以保存变量，我们可以用它来实现缓存，先定义一个计算斐波那切数列的函数，并加上上面定义的`counter`装饰器用来统计计算次数

```python
@counter
def fib(n):
    if n < 2:
        return n
    return fib(n-1) + fib(n-2)
```

计算`fib(10)`需要调用177次，而再次调用`fib(20)`就已经上升到了22086次，也就是说`fib(20)`单独调用需要21891次，这样的递归调用是非常糟糕的

```python
>>> fib(10)
fib call 10 times
fib call 11 times
fib call 11 times
fib call 12 times
...
fib call 177 times
55
>>>
>>> fib(20)               # long time
...
ib call 22086 times       # 22086 - 177 = 21891
6765
```

用装饰器实现一个cache

```python
def cache(f):
    @functools.wraps(f)
    def wrapper(*args, **kwargs):
        key = args + tuple(kwargs.items())
        if not wrapper.dict.get(key):
            wrapper.dict[key] = f(*args, **kwargs)
        return wrapper.dict[key]
    wrapper.dict = dict()
    return wrapper

@cache
@counter
def fib(n):
    if n < 2:
        return n
    return fib(n-1) + fib(n-2)
```

试验以下

```python
>>> fib(20)
fib call 20 times
...
fib call 21 times
fib call 21 times
6765
>>> fib(20)
6765
>>> fib(10)
55
```

发现第一次调用`fib(20)`后再次调用`fib(20)`或者小于20的数都不需要在次计算了，非常的快

> Python官方[`functools.lru_cache`](https://docs.python.org/3/library/functools.html#functools.lru_cache)已经内建可以使用，且功能更丰富

### 单例(Singletons)

可以使用装饰器来实现单例

```python
from functools import update_wrapper

class Singleton:
    def __init__(self, cls):
        update_wrapper(self, cls)
        self.instance = None
        self.cls = cls

    def __call__(self, *args, **kwargs):
        if not self.instance:
            self.instance = self.cls(*args, **kwargs)
        return self.instance

@Singleton
class Foo:
    pass

f0 = Foo()
f1 = Foo()
print(f0 is f1)   # True
print(id(f0))     # 4535987168
print(id(f1))     # 4535987168
```

### 字段验证

在api请求中通常约定特定字段作为接收参数，我们可以通过装饰器来验证改字段是否存在

```python
from flask import Flask, request, abort
from functools import wraps

app = Flask(__name__)

def validate(*json_keys):
    def decorator(f):
        @wraps(f)
        def wrapper(*args, **kwargs):
            if request.is_json:
                for k in json_keys:
                    if k not in request.get_json():
                        abort(400)
            return f(*args, **kwargs)
        return wrapper
    return decorator

@app.route("/info", methods=["POST"])
@validate("name", "age")
def user_info():
    return "Success"
```
上面的Flask应用，路由`/info`使用了`validate`装饰器检查`name`,`age`是否存在于请求json中，如果不存在返回400。定义装饰器时使用了`*json_keys`可以接收任意个key。

## Reference

- [www.python-course.eu](https://www.python-course.eu/python3_memoization.php)
- [realpython.com](https://realpython.com/primer-on-python-decorators/)
