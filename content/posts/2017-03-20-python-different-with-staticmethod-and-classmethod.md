---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2017-03-20T06:32:24+08:00'
description: Python中的装饰器@staticmethod和@classmethod的区别
lastmod: '2017-11-29T02:54:23+08:00'
showToc: true
tags: [Python]
title: Python类方法与静态方法的区别 | different between staticmethod and classmethod
---

Python中的装饰器@staticmethod和@classmethod的区别
====

### staticmethod

当不想访问类变量和实例变量，又想优雅地写代码，方法不写在类外面避免以后代码难以维护。可以这样写封装在类里面

    class TestStaticMethod(object):
        a = 0
        def __init__(self):
            TestStaticMethod.a += 1
        @staticmethod
        def smethod():
            print('static method')
    
    bar = TestStaticMethod()
    bar.smethod()
    TestStaticMethod.smethod()

输出

    static method
    static method
    [Finished in 0.0s]

静态方法`@staticmethod`即不能访问类变量也不能访问实例变量，被装饰的方法不需要传入任何参数。类和实例都可以调用。

### classmethod

在类中需要用到类变量而不需要实例参与的可以这样写

    class TestClassMethod(object):
        a = 0
        def __init__(self):
            TestClassMethod.a += 1
        @classmethod
        def cmethod(cls):
            print('class method')
            # 访问a
            print(cls.a)
    
    foo = TestClassMethod()
    foo.cmethod()
    TestClassMethod.cmethod()

输出

    class method
    1
    class method
    1
    [Finished in 0.0s]

类方法可以引用类变量，但被装饰的方法需要传入类对象参数`cls`，类和实例都可以调用。

### 总结

    class TestClassAndStaticMethod(object):
        @staticmethod
        def smethod(*args):
        	print('staticmethod', args)
        @classmethod
        def cmethod(*args):
            print('classmethod', args)
    
    foo = TestClassAndStaticMethod()
    TestClassAndStaticMethod.smethod()
    foo.smethod()
    TestClassAndStaticMethod.cmethod()
    foo.cmethod()

输出

    staticmethod ()
    staticmethod ()
    classmethod (<class '__main__.TestClassAndStaticMethod'>,)
    classmethod (<class '__main__.TestClassAndStaticMethod'>,)
    [Finished in 0.0s]

`@staticmethod`可以不传任何变量在封装在类中，`@classmethod`需传入`cls`变量代表类对象，可以引用类变量，避免 hard-coded。

***

**参考：**

[http://pythoncentral.io](http://pythoncentral.io/difference-between-staticmethod-and-classmethod-in-python/)

[http://stackoverflow.com](http://stackoverflow.com/questions/12179271/meaning-of-classmethod-and-staticmethod-for-beginner)