---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: '2019-12-17T11:38:45+08:00'
lastmod: '2020-04-17T14:06:42+08:00'
showToc: true
tags: [Python]
title: Python描述符(descriptor)
---

# Python描述符(descriptor)

Python中有一个很少被使用或者用户自定义的特性，那就是描述符(descriptor)，但它是`@property`, `@classmethod`, `@staticmethod`和`super`的底层实现机制，我今天就扒一扒它，[官方文档](https://docs.python.org/3/reference/datamodel.html#invoking-descriptors)对描述符的介绍如下

> In general, a descriptor is an object attribute with “binding behavior”, one whose attribute access has been overridden by methods in the descriptor protocol: `__get__()`, `__set__()`, and `__delete__()`. If any of those methods are defined for an object, it is said to be a descriptor.

描述符是绑定了行为的**对象属性(object attribute)**，实现了描述符协议(descriptor protocol)，描述符协议就是定义了`__get__()`,`__set__()`,`__delete__()`中的一个或者多个方法，将描述符对象作为其他对象的属性进行访问时，就会产生一些特殊的效果。

上面的定义可能还是有些晦涩，一步步来

## 默认查找属性

在没有描述符定义情况下，我们访问属性的顺序如下，以`a.x`为例

1. 查找实例字典里的属性就是`a.__dict__['x']`有就返回
2. 往上查找父类的字典就是`a.__class__.__dict__['x']`有就返回
3. 上面都没有就查找父类的基类(不包括元类(metaclass))
4. 如果定义了`__getattr__`就会返回此方法
5. 最后都没有抛出`AttributeError`

```python
>>> class A:
...     x = 8
...     
... 
>>> class B(A):
...     pass
... 
>>> class C(B):
...     def __getattr__(self, name):
...         if name == 'y':
...             print("call getattr method")
...         else:
...             raise AttributeError
...         
...     
... 
>>> C.__mro__
(<class '__main__.C'>, <class '__main__.B'>, <class '__main__.A'>, <class 'object'>)
>>> a = C()
>>> a.x
8
>>> a.y
call getattr method
>>> a.__dict__
{}
>>> a.x = 99
>>> a.x
99
>>> a.__dict__
{'x': 99}
```

> `__getattr__`是实例访问没有定义的属性时调用的方法，[需要特别定义](https://www.fythonfang.com/blog/2018/1/28/python-magical-special-methods#__getattr__self-name)

## 描述符协议

`object.__get__(self, instance, owner=None)`

- 在访问属性时被调用
- `self`是描述符本身，`instance`是使用描述符的实例，`owner`是使用描述符的类。
- 这里调用要分为类属性的调用(调用`owner`上)和实例对象属性(`instance`上)的调用。当调用类属性的时候`instance=None`。
- 返回值或者`AttributeError`

`object.__set__(self, instance, value)`

- 在属性赋值时被调用
- `value`为赋的值
- 无返回值

`object.__delete__(self, instance)`

- 在属性被删除时调用
- 无返回值

`object.__set_name__(self, owner, name)`

- 在`owner`类创建时被调用，给描述符命名，python3.6新增
- `name`为使用描述符的类的类属性的名字
- 无返回值

某个类只要定义了以上方法的一个或者多个就是实现了描述符协议，在作为某个对象属性时就是描述符，从而这个对象属性被重写默认的查找行为(上文所述)。描述符分*数据描述符(data descriptors)*和*非数据描述符(non-data descriptors)*，**仅**定义了`__get__`方法的叫非数据描述符，其它情况都是数据描述符，一般定义了`__get__`和`__set__`方法。数据描述符和非数据描述符对属性的查找顺序影响很大

当访问访问属性时，如`a.x`，`a`为实例访问`x`属性，如果`x`是描述符就不再遵守默认的查找行为，看情况优先级如下

- 如果`a`中的实例字典有同名的`x`描述符，且为数据描述符，则数据描述符优先访问
- 如果`a`中的实例字典有同名的`x`描述符，且为非数据描述符，则实例字典里面的优先访问

所以在有描述符的情况下实例属性的查找顺序：**数据描述符 > 实例字典 > 非数据描述符**


## 描述符实例

有了上面的理论，我们来看实例

### 数据描述符（Data Descriptors）

```python
class DataDescriptor:
    """A data descriptor that sets and returns values
       normally and prints a message logging their access.
    """
    def __init__(self, initval):
        self.initval = initval

    def __get__(self, instance, owner):
        print(f"get ... instance: {instance!r}, owner: {owner!r}")
        return self.initval

    def __set__(self, instance, value):
        print(f"set ... instance: {instance!r}, value: {value!r}")
        self.initval = value
```

以上`DataDescriptor`定义了`__get__`和`__set__`方法，当用作一个对象属性时就是一个数据描述符

```python
>>> class Person:
...     age = DataDescriptor(10)
...     
... 
>>> p = Person()
>>> p.__dict__
{}
>>> p.age
get ... instance: <__main__.Person object at 0x110a68590>, owner: <class '__main__.Person'>
10
>>> p.age = 18
set ... instance: <__main__.Person object at 0x110a68590>, value: 18
>>> p.__dict__
{}
>>> p.__dict__['age'] = 100
>>> p.age
get ... instance: <__main__.Person object at 0x110a68590>, owner: <class '__main__.Person'>
18
```

`DataDescriptor(10)`对象(`age`)就是一个数据描述符，根据上文的优先级实例字典是不会对它产生影响的所以`p.age`还是返回`18`

需要注意的是描述符作用在对象属性(类属性)上才是描述符，也就是说不能定义在`__init__`方法下

```python
>>> class Person:
...     age = DataDescriptor(10)
...     
...     def __init__(self):
...         self.weight = DataDescriptor(50)
...         
...     
... 
>>> p = Person()
>>> p.weight
<__main__.DataDescriptor object at 0x1085a2250>
```
上面`age`是描述符，`weight`不是。访问`p.weight`属性只返回`DataDescriptor`的实例对象

还有一个问题是`age`其实是一个类属性，`Person`的所有实例共享`age`这个实例变量，任何一个实例修改会导致所有的实例都更改。具体参看[Python中的类变量(class variables)和实例变量(instance variables)](https://www.fythonfang.com/blog/2017/3/25/difference-between-class-and-instance-variables)

```python
>>> class Person:
...     age = DataDescriptor(10)
...     
... 
>>> p1 = Person()
>>> p2 = Person()
>>> p1.age
get ... instance: <__main__.Person object at 0x10817b090>, owner: <class '__main__.Person'>
10
>>> p2.age
get ... instance: <__main__.Person object at 0x108159210>, owner: <class '__main__.Person'>
10
>>> p1.age = 18
set ... instance: <__main__.Person object at 0x10817b090>, value: 18
>>> p1.age
get ... instance: <__main__.Person object at 0x10817b090>, owner: <class '__main__.Person'>
18
>>> p2.age
get ... instance: <__main__.Person object at 0x108159210>, owner: <class '__main__.Person'>
18
```

`p1.age`更改后`p2.age`的值也随之改变了，可以使用一个字典存储每个实例对应的值

```python
from weakref import WeakKeyDictionary

class DataDescriptor:
    def __init__(self, default):
        self.default = default
        self.data = WeakKeyDictionary()

    def __get__(self, instance, owner):
        return self.data.get(instance, self.default)

    def __set__(self, instance, value):
        if value < 0:
            raise ValueError(f"Negative value not allowed: {value}")
        self.data[instance] = value
```

这样确保了每个实例对应的值都相互不影响，这里使用了弱引用字典防止内存爆表。还在赋值的时候做了非负检查

```python
>>> class Person:
...     age = DataDescriptor(1)
...     
... 
>>> p1 = Person()
>>> p2 = Person()
>>> p1.age
1
>>> p2.age
1
>>> p1.age = 18
>>> p1.age
18
>>> p2.age
1
```

最后一个问题就是正因为用的是字典存储专属于实例的数据，特殊情况是如果实例对象(`instance`)不可哈希，那就会报错

```python
>>> class MyList(list):
...     x = DataDescriptor(10)
...     
... 
>>> m = MyList()
>>> m.x
Traceback (most recent call last):
...
TypeError: unhashable type: 'MyList'
```

`Mylist`继承自`list`，所以传入的实例`instance`是不可哈希的，一个解决办法就是每次使用描述符的时候给它取个名字加标签

```python
class DataDescriptor:
    def __init__(self, default, name):
        self.default = default
        self.name = name

    def __get__(self, instance, owner):
        return instance.__dict__.get(self.name, self.default)

    def __set__(self, instance, value):
        if value < 0:
            raise ValueError(f"Negative value not allowed: {value}")
        instance.__dict__[self.name] = value

class MyList(list):
    x = DataDescriptor(1, 'x')

m = MyList()
print(m.x)     # 1
m.x = 8
print(m.x)     # 8
```

用一开始传入的`name`作为键，就避免了有可能键是不可哈希的问题，另一方面此方法涉及到每个实例的字典`__dict__`，因为这是一个数据描述符访问属性的时候优先调用`__get__`或者`__set__`方法，查找顺序优先于实例字典，然后我们在方法里面可以安全的访问对象的实例字典`instance.__dict__`，这有点绕但没有问题。把值存储在各对象的实例字典里面即解决不同实例相互影响问题又解决内存问题。但每次传`name`会有点麻烦可不可以不传呢，python3.6中对描述符协议新增了`__set_name__`特殊方法可以轻松获取描述符的名字，所以也可以这么写

```python
class DataDescriptor:
    def __init__(self, default):
        self.default = default

    def __get__(self, instance, owner):
        return instance.__dict__.get(self.name, self.default)

    def __set__(self, instance, value):
        if value < 0:
            raise ValueError(f"Negative value not allowed: {value}")
        instance.__dict__[self.name] = value

    def __set_name__(self, owner, name):
        print(f"set name called name: {name!r}")
        self.name = name
```

`__set_name__`方法会在类属性定义的时候被调用，获取名字(`x`)

```python
>>> class MyList(list):
...     x = DataDescriptor(10)
...     
... 
set name called owner: <class '__main__.MyList'>, name: 'x'
>>> m1 = MyList()
>>> m2 = MyList()
>>> m1.x
10
>>> m2.x
10
>>> m1.x = 99
>>> m1.x
99
>>> m2.x
10
>>> m1.__dict__
{'x': 99}
```

以上`DataDescriptor`可以在任何对象上使用，并且不受多个实例相互影响了。

### 非数据描述符（Non-Data Descriptors）

再来看一个非数据描述符

```python
class NonDataDescriptor:
    """A non-data descriptor
    """
    def __init__(self, initval):
        self.initval = initval

    def __get__(self, instance, owner):
        print(f"get ... instance: {instance!r}, owner: {owner!r}")
        return self.initval
```

只定义一个`__get__`方法的为非数据描述符

```python
>>> class Student:
...     age = NonDataDescriptor(13)
...     
... 
>>> s = Student()
>>> s.age
get ... instance: <__main__.Student object at 0x1109c3e10>, owner: <class '__main__.Student'>
13
>>> s.__dict__
{}
>>> s.age = 18
>>> s.__dict__
{'age': 18}
>>> s.age
18
>>> Student.age
get ... instance: None, owner: <class '__main__.Student'>
13
```
可以看出非数据描述符的优先级比实例字典低，赋值会存放到`__dict__`中，也是这个原因如果有多个实例相互之间赋值也不影响，不需要像上面那样单独为每个实例保存一份值，`Student.age`访问的是类变量所以`instance`为`None`

## 描述符的调用

访问属性时`obj.d`，如果`d`是描述符定义了`__get__`方法，要分两种情况因为`obj`可以是类或者实例，也就是说`obj.d`可能是类属性或者实例属性

- 对于`obj`是实例时，底层调用`object.__getattribute__()`实现，把`obj.b`转化成`type(obj).__dict__['b'].__get__(obj, type(obj))`
- 对于`obj`是类时，调用`object.__getattribute__()`时，如把`Cls.b`转化成`Cls.__dict__['b'].__get__(None, Cls)`

## 描述符的创建

有多个方式可以创建描述符

- 通过使用[`property()`](https://docs.python.org/3/library/functions.html#property)创建
- 创建一个类并实现描述符协议

### 通过使用`property()`创建

python提供了[`property()`](https://docs.python.org/3/library/functions.html#property)函数，可以用来创建描述符

```python
class Person:
    def __init__(self, initval):
        self._x = initval

    def get_x(self):
        print("get ...")
        return self._x

    def set_x(self, value):
        print("set ...")
        self._x = value

    def del_x(self):
        print("del ...")
        del self._x

    age = property(get_x, set_x, del_x, "I'm the 'age' property.")
```

类`Person`定义了`age`属性，其实`age`就是一个描述符

```python
>>> Person.age
<property object at 0x10310b290>
>>> p = Person(10)
>>> p.age
get ...
10
>>> p.__dict__
{'_x': 10}
>>> del p.age
del ...
>>> p.age = 18
set ...
>>> p.__dict__
{'_x': 18}
>>> p.age
get ...
18
```

此方法可以看到`age`是`property object`，`property()`函数实现为*数据描述符*。因此，实例字典是无法覆盖的(`name`不在`__dict__`中)，但从上面发现其实我们引入了`_x`私有变量。这种方法对某个属性的定义非常好用，python还特地提供了语法糖`@property`写起来更加方便，以前文章也有介绍 [Python中@propery 使用](https://www.fythonfang.com/blog/2017/10/8/python-property-tutorial)

```python
class Person:
    def __init__(self, initval):
        self.__age = initval

    @property
    def age(self):
        print("get ...")
        return self.__age

    @age.setter
    def age(self, value):
        print("set ...")
        self.__age = value

    @age.deleter
    def age(self):
        print("del ...")
        del self.__age
```

### 创建一个类并实现描述符协议

创建一个类并覆盖任意一个描述符方法`__set__`、`__ get__` 、 `__delete__`和`__set_name__`，之前我们创建的`DataDescriptor`和`NonDataDescriptor`都是用的此方法，当需要某个属性在多个不同的类或者实例都可以使用时，例如类型验证，值检查，都可以使用该方法创建。

试想如果我们需要类型验证很多的属性用上述`@property`的方法就写起来比较繁琐了要写多个`@property`块定义，用此方法就很简单，如

```python
class Foo:
    a = DataDescriptor(1)
    b = DataDescriptor(2)
    ....
```

## 实际使用

### 只读属性和惰性求值

```python
class ReadonlyNumber(object):
    """
    实现只读属性(实例属性初始化后无法被修改)
    利用了 data descriptor 优先级高于 obj.__dict__ 的特性
    当试图对属性赋值时，总会先调用 __set__ 方法从而抛出异常
    """
    def __init__(self, value):
        self.value = value

    def __get__(self, instance, owner):
        return self.value

    def __set__(self, instance, value):
        raise AttributeError(
            "'%s' is not modifiable" % self.value
         )


class LazyProperty(object):
    """
    实现惰性求值(访问时才计算，并将值缓存)
    利用了 obj.__dict__ 优先级高于 non-data descriptor 的特性
    第一次调用 __get__ 以同名属性存于实例字典中，之后就不再调用 __get__
    """
    def __init__(self, fun):
        self.fun = fun

    def __get__(self, instance, owner):
        if instance is None:
            return self
        value = self.fun(instance)
        setattr(instance, self.fun.__name__, value)
        return value


class Circle(object):

    pi = ReadonlyNumber(3.14)

    def __init__(self, radius):
        self.radius = radius

    @LazyProperty
    def area(self):
        print('Computing area')
        return self.pi * self.radius ** 2

y = Circle(3)
y.area             # 28.26
```

`ReadonlyNumber`描述符实现了只读属性，`LazyProperty`实现了属性值缓存这里用到了[装饰器](https://www.fythonfang.com/blog/2017/4/10/python-decorators)

### 函数与方法

上面我们已经看到`property`是一个数据描述符。接下来我们看看函数。

类中的函数就是方法，其实函数就是一个*非数据描述符*只定义了`__get__()`方法，所以能被实例字典覆盖

```python
>>> class D:
...     def f(self, x):
...         return x
...     
... 
>>> d = D()
>>> D.__dict__['f']             # 通过类字典访问f，不调用__get__
<function D.f at 0x108b17e60>
>>> D.f                         # 通过类属性访问，调用__get__
<function D.f at 0x108b17e60>
>>> D.__dict__['f'].__get__(None, D)  # 手动调用__get__方法
<function D.f at 0x108b17e60>
>>> D.f.__qualname__
'D.f'
>>> d
<__main__.D object at 0x108486710>
>>> d.f                         # 实例属性调用__get__，返回bound method
<bound method D.f of <__main__.D object at 0x108486710>>
>>> type(d).__dict__['f'].__get__(d, type(d))  # 手动调用
<bound method D.f of <__main__.D object at 0x108486710>>
# 绑定的方法内部存储了函数地址、绑定此方法的实例、以及绑定实例的类
>>> d.f.__func__                # 函数
<function D.f at 0x108b17e60>
>>> d.f.__self__                # 实例对象
<__main__.D object at 0x108486710>
>>> d.f.__class__               # 类
<class 'method'>
>>> d.f = 100
>>> d.f
100
```

我们知道类方法就是定义在类内部的函数只是第一个参数(`self`)接收自身实例对象，当使用dot notation(`.`)访问时，把实例对象传给第一个参数。因为函数`f`是一个非数据描述符，当调用`d.f(*args)`时，内部的`__get__`方法会把`d.f(*args)`转化成`f(d, *args)`，当调用`D.f(*args)`是转化成`f(*args)`，这就是非数据描述符干的事情。

### 静态方法和类方法

没错静态方法和类方法也是和上面函数调用同样的原理，如类方法调用（从类调用）内部`__get__`就是把`OneClass.f(*args)`转化成`f(OneClass, *args)`，静态方法同理，官方文档提供了如下的转化表格

|  转型   |     从实例对象调用      |      从类调用       |
| ------- | -------------------- | ------------------ |
| 函数    | `f(ojb, *args)`       | `f(*args)`         |
| 静态方法 | `f(*args)`            | `f(*args)`         |
| 类方法   | `f(type(obj), *args)` | `f(kclass, *args)` |

## 小结

1. 描述符要实现描述符协议（实现`__set__`, `__get__`, `__delete__`, `__set_name__`方法）
2. 描述符必须作为对象属性（类属性）
3. 描述符的查找顺序：数据描述符 > 实例字典 > 非数据描述符


## Reference

1. [docs.python.org](https://docs.python.org/3/howto/descriptor.html)
2. [realpython.com](https://realpython.com/python-descriptors/)
3. [www.ibm.com](https://www.ibm.com/developerworks/cn/opensource/os-pythondescriptors/index.html)
4. [www.jianshu.com](https://www.jianshu.com/p/fe66aebc02ec)
5. [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/42485483)
