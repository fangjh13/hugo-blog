---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: "2018-01-28T05:36:12+08:00"
description: 整理类定义时的特殊方法
lastmod: "2020-04-21T07:57:04+08:00"
showToc: true
tags: [Python]
title: Python中的特殊方法
---

# Python 中的特殊方法

python 在定义 class 时有很多特殊方法可以定义，它们一般都是以双下划线开头和结尾如`__init__`、`__call__`、`__lt__`、`__iter__`、`__setattr__`、`__setitem__`等，下面将对这些常用方法作一些总结。

## class 基本方法

### `__new__(cls[, ...])`

我们都知道`__init__`会在类初始化的时候被调用其实它不是第一个被调用的，第一个是`__new__`会在对象创建的时候被调用，这时还没有实例化所以它的第一个参数是`cls`。

### `__init__(self[, ...])`

最常用的类方法不用过多介绍，对象初始化时自动调用，也就是在`__new__`之后才会调用。第一个参数是`self`实例本身。有一点要注意的是它不能返回(`return`)值，不然会报`TypeError`记住它是用来初始化对象的。

```python
class A:
    def __new__(cls):
        print('__new__ called')
        return super(A, cls).__new__(cls)

    def __init__(self):
        print('__init__ called')

a = A()
```

返回如下，注意这是 Python3 的写法默认`A`继承自`object`，如果使用 Python2 的需要指定`A`继承自`object`基类，使用新类。不然`__new__`是不会被调用的。

```shell
__new__ called
__init__ called
```

需要注意的是上面我们定义了`__new__`接收类对象(`cls`)，必须返回`return super(A, cls).__new__(cls)`这个类实例也就是下面方法的`self`，因为`__new__`会被首先调用返回实例对象以供下面的方法如`__init__`这些使用。如果去掉这一句其他方法会返回`None`。默认未定义`__new__`方法时，默认返回父类实例`super().__new__(cls, *args, **kwargs)`

### `__str__(self)`

调用`print(obj)`或`str(obj)`时返回的字符串是对人类友好的(human readable string)。

### `__repr__(self)`

调用`repr(obj)`或`obj`返回的字符串，比较偏向机器。

### `__format__(self, format_spec)`

调用`f string`或`format`时调用，`format(value, format_spec)`相当于`value.__format__(format_spec)`

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def __format__(self, format_spec):
        if format_spec == 'view':
            return f"this is {self.name}"
        else:
            raise ValueError('invalid format spec.')

dog = Animal('Labrador')

print(format(dog, "view"))
print("{0:view}".format(dog))
print(f"{dog:view}")
```

三种方式返回都一样如下

```shell
this is Labrador
this is Labrador
this is Labrador
```

### `__bytes__(self)`

调用`bytes(obj)`时返回，返回对象必须是`bytes`

### `__bool__(self)`

调用`bool(obj)`时返回，返回值是`True`或者`False`，在 python2 中方法名为`__nonzero__`。如果调用`bool(obj)`时，没有定义`__boo__`时会会去调用`__len__`方法返回非零为`True`，如果这两个方法都没有定义那么默认返回`True`。

## 比较(comparison)

### `__lt__(sel, other)`

小于

### `__le__(self, other)`

小于等于

### `__eq__(self, other)`

等于

### `__ne__(self, other)`

不等于

### `__gt__(self, other)`

大于

### `__ge__(self, other)`

大于等于

Python2 中有`__cmp__`方法和`cmp()`函数，在 Python3 中已经废弃没有了代替的是以上 6 个方法分别对应`<`，`<=`，`==`，`!=`，`>`，`>=`操作符时返回的值，通常情况下要求返回`True`或`False`，但其实可以返回任何值。在`if`语句中如果返回非`True`或`False`，会自动对返回调用`bool`来判断。

## 哈希

### `__hash__(self)`

调用`hash(obj)`时返回的值，是一个整数。当两个对象是相等(`==`)时，他们的 hash 值也必定相等也就是`hash(obj) == hash(obj)`为`True`。

> 使用**hash**时记得定义**eq**方法

## 描述符(descriptor)

> 描述符属于 py 中比较高级的主题，单独记录了一篇[文章](../2019-12-17-python-descriptor/)，以下只列出特殊方法

### `__get__(self, instance, owner=None)`

用于访问属性，返回属性的值。

### `__set__(self, instance, value)`

将在属性分配操作中调用。不会返回任何内容。

### `__delete__(self, instance)`

控制删除操作。不会返回内容。

### `__set_name__(self, owner, name)`

Python3.6 中对描述符新增的方法，在`owner`创建时被调用，给描述符命名。

## 属性(attribute)

### `__getattr__(self, name)`

当访问属性没有按正常检索顺序检索到时被调用的方法。

```python
class A:
    def __getattr__(self, name):
        if name == 'foo':
            return 'foo attribute'
        else:
            raise AttributeError

a= A()
print(a.foo)
a.foo = 8   #正常赋值
print(a.foo)
print(a.bar)
```

返回如下，只有在没有被检索到时才会被调用。

```python
foo attribute
8
Traceback (most recent call last):
  File "/Users/fython/Documents/testDemo/test.py", line 12, in <module>
    print(a.bar)
  File "/Users/fython/Documents/testDemo/test.py", line 6, in __getattr__
    raise AttributeError
AttributeError
```

### `__getattribute__(self, name)`

无论访问存在或者不存在的属性时都会被调用，即使访问对象方法时也会被调用，所以这个是无条件优先级最高的使用这个方法要十分小心防止进入无限循环调用

```python
class A:
    def __init__(self):
        self.foo = 'foo'

    def __getattribute__(self, name):
        return self.__dict__[name]

    def __getattr__(self, name):
        return 'default'

    def bar(self):
        return 'test normal method'

a= A()
print(a.foo)  # 访问存在的属性
print(a.fake_foo)  # 访问不存在的属性
print(a.bar())  # 调用方法
```

如上就会掉入`__getattribute__`的陷阱，以上三种调用方式都会引发无限循环引用`RecursionError`。因为`__getattribute__`第一个被调用返回`self.__dict__[name]`，这就等于又去访问`self`的`__dict__`属性，又会陷入`__getattribute__`方法，循环不断。。。。就算定义了`__getattr__`方法也是会被忽略。值得注意的是调用其他方法`a.bar()`也会出问题。那么如何解决呢，一般都是调用基类相同方法就可以，可以用`super`实现。

```python
class A:
    ...
    def __getattribute__(self, name):
        if name == 'fake_foo':
            return 'fake_foo'
        else:
            return super(A, self).__getattribute__(name)
    ...
```

改成以上的运行就没有什么问题

```python
foo
fake_foo
test normal method
```

### `__setattr__(self, name, value)`

在对象属性被赋值时被调用的方法，这也容易引发无限循环调用

```python
class A:
    def __setattr__(self, name, value):
        self.name = value

a= A()
a.foo = 'foo' # 无限循环
```

和之前一样`self.name = value`又会调用`__setattr__`，解决方法也是一样换成`super(A, self).__setattr__(name, value)`

```python
class A:
    def __setattr__(self, name, value):
        if isinstance(value, str):
            return super(A, self).__setattr__(name, value + '_suffix')
        else:
            raise TypeError

a= A()
a.foo = 'foo'
print(a.foo)  # 返回"foo_suffix"
```

### `__delattr__(self, name)`

属性删除，一样有循环引用的问题(`return del self.name`)，记得用`super()`解决。

## 类创建

### `__init_subclass__(cls)`

这是 python3.6 中新加的方法，它会在以这个类为基类创建子类时被调用。`cls`是新的子类。

```python
class Philosopher:
    def __init_subclass__(cls, default_name, **kwargs):
        super().__init_subclass__(**kwargs)
        cls.default_name = default_name

class AustralianPhilosopher(Philosopher, default_name="Bruce"):
    pass

print(AustralianPhilosopher.default_name)
```

`AustralianPhilosopher`继承自`Philosopher`而后者定义了`__init_subclass__`，此方法接收子类定义时继承的参数。

## 可调用对象(callable objects)

### `__call__(self[,args...])`

使实例对象可以像函数一样调用，`x(arg1, arg2, ...)`就等于调用`x.__call__(arg1, arg2, ...)`

```python
class A:
    def __call__(self, *args, **kwargs):
        print(args, kwargs)

a = A()
a('a', 'b', c='d')  # 可被调用  输出('a', 'b') {'c': 'd'}
```

## 容器(container)类型

容器类代表的是 sequences(list, tuples)，mappings(dict, set)

### `__len__(self)`

当调用`len()`时会调用，应该返回对象的长度是一个整数。

### `__getitem__(self, key)`

当调用`self[key]`时会调用，对于 sequences 类来说 key 是索引值

### `__setitem__(self, key, value)`

当`self[key]`被赋值时会调用。

### `__delitem__(self, key)`

当`del self[key]`时被调用

### `__missing__(self, key)`

当调用`self[key]`时`__gititem__`字典中间值不存在时返回的值

```python
class A:
    def __setitem__(self, key, value):
        if key == 'foo':
            self.__dict__[key] = 'value'
        else:
            self.__dict__[key] = value

    def __getitem__(self, key):
        if key == 'bar':
            return 'custom value'
        else:
            return self.__dict__[key]

    def __delitem__(self, key):
        del self.__dict__[key]

a = A()
a['foo'] = 1234  # 调用__setitem__
print(a['foo'])  # "value"
print(a['bar'])  # "custom value"
del a['foo']
print(a['foo'])  # rasie KeyError
```

### `__reversed__(self)`

当调用`reversed()`时被调用，返回一个与之前相反的序列列表。

### `__contains__(self, item)`

当调用`in`或`not in`时返回的值，检测元素或者 key 是否存在于列表或字典中。返回`True`或`False`。

### `__iter__(self)`

当定义了一个`__iter__`方法后，这个对象就是可迭代的(iterable)，也就是说可以用于`for`这种循环。该方法应该返回一个新的可迭代对象(函数`iter`可生成)。

### `__next__(self)`

当一个对象即定义了`__iter__`又定义了`__next__`方法后，我们称它为迭代器(iterator)，注意与上面 iterable 的区别。本质上是 iterator 可以使用`next(obj)`来调用而 iterable 对象不可以。但双方都可以用`for`循环迭代。

```python
class MyIterator:
    def __init__(self, letters):
        self.letters = letters
        self.position = 0

    def __iter__(self):
        return self  # 注意这里

    def __next__(self):
        if self.position > len(self.letters) - 1:
            raise StopIteration
        rs = self.letters[self.position]
        self.position += 1
        return rs

m = MyIterator('abcdefg')
for i in m:
    print(i, end=', ')  # a, b, c, d, e, f, g,

n = MyIterator('xy')
print(next(n))  # x
print(next(n))  # y
print(next(n))  # raise StopIteration
```

上面就是一个迭代器同时定义了`__iter__`和`__next__`方法。注意两个都定义时在`__iter__`中只要返回`self`本身就可以了。

### `__reversed__(self)`

当调用`reversed()`时被调用，返回一个与之前相反的序列列表。

## 用户定义的函数（functions）

### `__doc__`

用户定义的文档，没有就返回`None`

### `__name__`

函数名

### `__closure__`

用户定义的函数都有一个`__closure__`属性，如果这个函数是一个闭包的话，那么它返回的是一个由`cell`对象组成的元组对象。`cell`对象的`cell_contents`属性就是闭包中的自由变量。

```python
def foo():
    a = 'hello world'
    b = 1.0
    def bar(c):
        return a, b, c
    return bar

x = foo()  # closure
print([i.cell_contents for i in x.__closure__])  # ['hello world', 1.0]
```

上面 x 就是一个闭包里面包含了`bar`函数和`a`,`b`变量。

## Reference

1. [http://spyhce.com/](http://spyhce.com/blog/understanding-new-and-init)
2. [https://docs.python.org/](https://docs.python.org/3/reference/datamodel.html)
3. [https://stackoverflow.com/](https://stackoverflow.com/questions/4295678/understanding-the-difference-between-getattr-and-getattribute)
4. [https://www.blog.pythonlibrary.org/](https://www.blog.pythonlibrary.org/2016/05/03/python-201-an-intro-to-iterators-and-generators/)
