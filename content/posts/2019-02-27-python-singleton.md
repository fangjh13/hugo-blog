---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2019-02-27T09:07:17+08:00'
lastmod: '2020-04-27T09:08:52+08:00'
showToc: true
tags: [Python]
title: Python实现单例(Singleton)的几种方法
---

# Python实现单例(Singleton)的几种方法

单例是一种比较简单的设计模式，每次实例化只提供一个相同的实例对象，对于保证实例唯一和节约系统资源的时候十分有用，下面就看看python中实现单例的几种方法

## 使用`__new__`方法

因为在类的实例化过程中`__new__`方法会比`__init__`提前调用，我们在类属性中保存一个`_singleton`每次只返回这个。

```python
class Singleton:
    def __new__(cls, *args, **kwargs):
        if not getattr(cls, '_singleton', None):
            cls._singleton = super().__new__(cls, *args, **kwargs)
        return cls._singleton


class MyClass(Singleton):
    pass

a = MyClass()
b = MyClass()
print(id(a))       # 4433117872
print(id(b))       # 4433117872
print(a is b)      # True
```

## 使用装饰器

```python
from functools import wraps

def singleton(cls):
    _singleton = {}
    @wraps(cls)
    def wrapper(*args, **kwargs):
        if not _singleton.get(cls):
            _singleton[cls] = cls(*args, **kwargs)
        return _singleton[cls]
    return wrapper

@singleton
class MyClass:
    pass
```

利用装饰器中的`_singleton`变量存储所有类的实例

## 利用python模块

python模块（module）是天然的单例模式

```python
# singleton.py

class Singleton:
    def foo(self):
        print("I'm singleton")

instance = Singleton()

del Singleton
```

然后利用模块导入

```python
from single import instance

instance.foo()  # I'm singleton
```