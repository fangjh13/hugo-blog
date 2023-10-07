---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2017-05-18T03:27:34+08:00'
description: 协程、asyncio、async/await
lastmod: '2018-04-25T06:06:49+08:00'
showToc: true
tags: [Python]
title: Python3 协程(Coroutine) 与 asyncio
---

## Python3 Coroutine(协程) 与 asyncio

> 协程，又称微线程，纤程，英文名Coroutine。协程的作用，是在执行函数A时，可以随时中断，去执行函数B，然后中断继续执行函数A（可以自由切换）。但这一过程并不是函数调用（没有调用语句），这一整个过程看似像多线程，然而协程只有一个线程执行。

**优势**

- 执行效率极高，因为子程序切换（函数）不是线程切换，由程序自身控制，没有切换线程的开销。所以与多线程相比，线程的数量越多，协程性能的优势越明显。
- 不需要多线程的锁机制，因为只有一个线程，也不存在同时写变量冲突，在控制共享资源时也不需要加锁，因此执行效率高很多。
 
*说明：协程可以处理IO密集型程序的效率问题，但是处理CPU密集型不是它的长处，如要充分发挥CPU利用率可以结合多进程加协程。*

有一篇*David Beazley*的课程[A Curious Course on Coroutines and Concurrency](http://www.dabeaz.com/coroutines/)详细讲解了协程和并发的用法，强烈推荐。本篇文章多处参考与此。

### 0x01. Generator与Coroutine的区别
一开始我总是傻傻分不清`Generator`和`Coroutine`的区别感觉这两个东西差不多不一样吗，最近查了点资料并学习了下。在此做记录，我们先来看`Generator`。

    def countdown(n):
    	while n > 0:
    		yield n
    		n -= 1

    c = countdown(5)
    print(c)

    for i in c:
    	print(i)

返回一个`generator object`并且可以迭代，详细参考[上一篇文章](http://www.fythonfang.com/blog/post/11)

    <generator object countdown at 0x7f82a41739e8>
    5
    4
    3
    2
    1
    [Finished in 0.0s]

如下是`Corountine`

    def coprint():
    	while True:
    		rs = yield
    		print(rs)

    cp = coprint()
    cp.send(None)
    cp.send(1)
    cp.send(2)
    cp.send(3)
 `send(None)`或者`next()`方法初始化一个协程。`send`可以传递一个对象给`cp`处理遇到`yield`停止等待下一个`send`并继续运行。`next()`方法或者`send`可以传递一个对象给`cp`处理遇到`yield`停止等待下一个`send`

对比可以发现虽然他们有很多相同之处但也是有区别的：

-  `generator`是生成器顾名思义它负责生产对象，也就是`yield`后跟着的值。
-  `generator`可以迭代用于`for`循环等。
- `coroutine`协程是一个消费者，消费`send`给协程的值（`yield`的左边）
- `coroutine`不能用于迭代，主要用于内部消费处理操作

### 0x02. 协程(Corountine)管道(Pipeline)的运用
先来实现一个使用协程版的`tail -f`命令。

    def follow(f, target):
	    f.seek(0, 2)
    	while True:
    	    last_line = f.readline()
    		if last_line is not None:
    			target.send(last_line)

    def printer():
    	while True:
    		line = yield
    		print(line, end='')


    f = open('access-log')
    prt = printer()
    next(prt)
    follow(f, prt)

以上只要`access-log`文件有写入`follow()`就会显示出来，而真正的协程是`printer()`并非`follow()`所以`prt`需要调用`next`方法初始化启动，而`follow`它只是一个调度者`send`数据给协程。

我们加入类似与`grep`过滤的功能。`filter`也是一个协程。但加了一个装饰器`init_coroutine`以便不用每次调用`next()`方法

     def init_coroutine(func):
     	def wrapper(*args, **kwargs):
     		rs = func(*args, **kwargs)
     		next(rs)
     		return rs
     	return wrapper

    def follow(f, target):
    	f.seek(0, 2)
    	while True:
    		last_line = f.readline()
    		if last_line is not None:
    			target.send(last_line)

    @init_coroutine
    def printer():
    	while True:
    		line = yield
    		print(line, end='')

    @init_coroutine
    def filter(key_string, target):
    	while True:
    		line = yield
    		if key_string in line:
    			target.send(line)

    f = open('access-log')
    follow(f, filter('python', printer()))

上面的效果等同于`tail -f access-log | grep python`，`filter`也是一个协程并且数据可以传递所以先`send`到`filter`然后再到`printer`显示出来。这就是协程的管道。你还可以像这样的`follow(f, filter('python', filter('string', printer())))`就两次过滤等于用了两次`grep`。

![](https://img.fythonfang.com/Screenshot%20from%202017-05-16%2019-24-22.png)

### 0x03. asyncio
`asyncio`是Python 3.4中新增的模块，是一个基于事件循环的实现异步I/O的模块，它提供了一种机制，使得你可以用协程（coroutines）、IO复用（multiplexing I/O）在单线程环境中编写并发模型。

    import asyncio

    @asyncio.coroutine
    def co_print(n):
    	print('Start: ', n)
    	r = yield from asyncio.sleep(1)
    	print('End: ', n)

    loop = asyncio.get_event_loop()
    tasks = [co_print(i) for i in range(5)]
    loop.run_until_complete(asyncio.wait(tasks))
    loop.close()

    > Out：
    Start:  3
    Start:  4
    Start:  1
    Start:  2
    Start:  0
    End:  3
    End:  4
    End:  1
    End:  2
    End:  0
    [Finished in 1.1s]

`@asyncio.coroutine`把一个`generator`标记为`coroutine`类型，从上面的运行结果可以看到每个`co_print(n)`是同时开始执行的，线程并没有等待前一个执行完再运行下一个。每个`co_print(n)`遇到`yield from`都会中断，并继续执行下一个`co_print(n)`。

    @asyncio.coroutine
    def compute(x, y):
    	print("Compute %s + %s ..." % (x, y))
    	yield from asyncio.sleep(1)
    	return x + y

    @asyncio.coroutine
    def print_sum(x, y):
    	result = yield from compute(x, y)
    	print("%s + %s = %s" % (x, y, result))

    loop = asyncio.get_event_loop()
    loop.run_until_complete(print_sum(1, 2))
    loop.close()

    > Out:
    Compute 1 + 2 ...
    # 停顿大约一秒
    1 + 2 = 3
    [Finished in 1.1s]

这是`coroutine`嵌套的例子，当事件循环(`EventLoop`)开始运行时，它会在Task中寻找coroutine来执行调度，因为事件循环注册了`print_sum()`，然后`print_sum()`被调用，执行`result = yield from compute(x, y)`这条语句，因为`compute()`自身就是一个`coroutine`，因此`print_sum()`这个协程就会暂时被挂起，`compute()`被加入到事件循环中，程序流执行`compute()`中的print语句，打印”Compute %s + %s …”，然后执行了`yield from asyncio.sleep(1.0)`，因为`asyncio.sleep()`也是一个`coroutine`，接着`compute()`就会被挂起，等待计时器读秒，在这1秒的过程中，事件循环会在队列中查询可以被调度的`coroutine`，而因为此前`print_sum()`与`compute()`都被挂起了没有其余的`coroutine`，因此事件循环会停下来等待协程的调度，当计时器读秒结束后，程序流便会返回到`compute()`中执行`return`语句，结果会返回到`print_sum()`中的`result`中，最后打印`result`，事件队列中没有可以调度的任务了，此时`loop.close()`把事件队列关闭，程序结束。

![](https://img.fythonfang.com/coroutine_chain.png)

### 0x04. one more thing
Python3.5中又添加了` async def`、`await`这样就使得协程变得更加易用了。[PEP 492](https://www.python.org/dev/peps/pep-0492/)中详细说明了使用`async`、`await`来定义`coroutine`避免和`generator`混淆。

只要把`@asyncio.coroutine`替换成`async`加在函数头，把`yield from`替换成`await`，其余不变就好了。但不能在同一个`coroutine`混用，就是用了`@asyncio.coroutine`而里面却用`yield from`中断。

    async def compute(x, y):
    	print("Compute %s + %s ..." % (x, y))
    	await asyncio.sleep(1)
    	return x + y

    async def print_sum(x, y):
    	result = await compute(x, y)
    	print("%s + %s = %s" % (x, y, result))

还有就是`await`之后必须接支持协程的函数或语句上面`asyncio.sleep(1)`就是一个模拟异步IO的过程，否者程序会同步执行看下面例子

```python
import time
import asyncio
import threading

async def normal_sleep(w, i):
    print('[{}] id ({}) normal sleep 2 seconds.'.format(w, i))
    time.sleep(2)
    print('[{}] id ({}) ok'.format(w, i))

async def worker(name):
    print('-> start {} worker {}'.format(name, threading.current_thread().name))
    for i in range(3):
        print('handle {}..'.format(i))
        await normal_sleep(name, i)
    print('<- end {}.\n'.format(name))

start = time.time()
loop = asyncio.get_event_loop()

# 启动两个worker
tasks = [worker('A'), worker('B')]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()

print('Total time {:.2f}s'.format(time.time()-start))
```

运行结果

```
-> start A worker MainThread
handle 0..
[A] id (0) normal sleep 2 seconds.
[A] id (0) ok
handle 1..
[A] id (1) normal sleep 2 seconds.
[A] id (1) ok
handle 2..
[A] id (2) normal sleep 2 seconds.
[A] id (2) ok
<- end A.

-> start B worker MainThread
handle 0..
[B] id (0) normal sleep 2 seconds.
[B] id (0) ok
handle 1..
[B] id (1) normal sleep 2 seconds.
[B] id (1) ok
handle 2..
[B] id (2) normal sleep 2 seconds.
[B] id (2) ok
<- end B.

Total time 12.02s
```

观察以上输出，发现程序共一个线程是串行执行的，就是因为使用了`time.sleep`，现在我们改成`asyncio.sleep(2)`

```python
...
async def async_sleep():
    print('async sleep 2 seconds.')
    await asyncio.sleep(2)
...
```

运行结果

```
-> start A worker MainThread
handle 0..
[A] id (0) normal sleep 2 seconds.
-> start B worker MainThread
handle 0..
[B] id (0) normal sleep 2 seconds.
[A] id (0) ok
handle 1..
[A] id (1) normal sleep 2 seconds.
[B] id (0) ok
handle 1..
[B] id (1) normal sleep 2 seconds.
[A] id (1) ok
handle 2..
[A] id (2) normal sleep 2 seconds.
[B] id (1) ok
handle 2..
[B] id (2) normal sleep 2 seconds.
[A] id (2) ok
<- end A.

[B] id (2) ok
<- end B.

Total time 6.01s
```

程序也共一个线程，当遇到`asyncio.sleep(1)`时会被挂起，`EventLoop`去处理另一个任务并等待返回结果，总的运行时间大大减小，异步非阻塞，这看起来像是多线程在执行，这就是协程的最大特点。
   
**参考**

[http://www.dabeaz.com](http://www.dabeaz.com/coroutines/)

[https://docs.python.org](https://docs.python.org/3/library/asyncio-task.html)

[http://www.liaoxuefeng.com](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001432090171191d05dae6e129940518d1d6cf6eeaaa969000)

[http://www.jianshu.com](http://www.jianshu.com/p/afa86801c038)

[http://thief.one](http://thief.one/2017/02/20/Python%E5%8D%8F%E7%A8%8B/)