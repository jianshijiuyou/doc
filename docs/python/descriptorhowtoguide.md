> [原文链接](https://docs.python.org/3/howto/descriptor.html)  
> [参考链接](https://www.jianshu.com/p/d08463f98714)

# 描述符使用指南

## 概要

定义描述符，总结协议，并展示如何调用描述符。审查自定义描述符和几个内置的 python 描述符，包括函数，属性，静态方法和类方法。通过给出一个纯 Python 等价物和一个示例来展示它们的工作原理。

学习描述符不仅可以访问更大的工具集，还可以深入了解 Python 的工作原理，并对其设计的优雅性有所了解。

## 定义和介绍

通常，描述符是具有「绑定行为」的对象属性，其属性访问已被描述符协议中的方法覆盖。这些方法是 `__get__()`，`__set__()`，和 `__delete__()`。 如果为某个对象定义了这些方法中的任何一种，则称它为描述符。

属性访问的默认行为是从对象的字典中获取，设置或删除属性。例如，`a.x` 的查找过程是：先找 `a.__dict__['x']` ，再找 `type(a).__dict__['x']`，然后遍历 `type(a)` 的基类。如果查找的值是定义其中一个描述符方法的对象，则 Python 可以覆盖默认行为并调用描述符方法。这种情况下调用的查找顺序取决于定义了哪些描述符方法。

描述符是一个强大的通用协议。它们是属性，方法，静态方法，类方法和 `super()` 背后的机制。它们被用于整个 Python 本身，以实现版本 2.2 中引入的新式类。描述符简化了底层 C 代码，为日常 Python 程序提供了一套灵活的新工具。

## 描述符协议

`descr.__get__(self, obj, type=None) --> value`

`descr.__set__(self, obj, value) --> None`

`descr.__delete__(self, obj) --> None`

这就是它的全部。定义这些方法中的任何一个，对象就被视为描述符，并且会在查找属性时覆盖默认行为。

如果一个对象同时定义了 `__get__()` 和 `__set__()`，它就被认为是一个数据描述符（也称覆盖型描述符或强制描述符）。只定义 `__get__()` 的描述符被称为非数据描述符（也称非覆盖型描述符或遮盖型描述符）（它们通常用于方法，但其他用途也是可能的）。

数据和非数据描述符在如何根据实例字典中的条目计算覆盖时有所不同。如果实例的字典中有一个与数据描述符同名的条目，则数据描述符优先。如果实例的字典中有一个与非数据描述符名称相同的条目，则字典条目优先。

要创建只读数据描述符，需要定义 `__get__()` 和 `__set__()`，并且在 `__set__()` 被调用时抛出 `AttributeError` 异常。用引发占位符的异常定义 `__set__()` 方法足以使其成为数据描述符。

## 调用描述符

描述符可以通过其方法名直接调用。例如，`d.__get__(obj)`。

不过，更为常见的是描述符在属性访问时被自动调用。例如，`obj.d` 会在 `obj` 的字典中查找 `d`。如果 `d` 定义了方法 `__get__()`，则根据下面列出的优先级规则调用 `d.__get__(obj)`。

调用的细节取决于 `obj` 是一个对象还是一个类。

对于对象来说，其中的机制在于 `object.__getattribute__()`，它将 `b.x` 转换成 `type(b).__dict__['x'].__get__(b, type(b))`。这个方法实现了一个优先级链，在该链中，数据描述符优先级高于实例变量，实例变量优先级高于非数据描述符，如果定义了 `__getattr__()`，那么它的优先级最低。

对类来说，其中的机制在于 `type.__getattribute__()` 将 `B.x` 转换成 `B.__dict__['x'].__get__(None, B)`。在纯 Python 中，它看起来像：

``` python
def __getattribute__(self, key):
    "Emulate type_getattro() in Objects/typeobject.c"
    v = object.__getattribute__(self, key)
    if hasattr(v, '__get__'):
        return v.__get__(None, self)
    return v
```

需要记住的重点是：

* 描述符由 `__getattribute__()` 方法调用
* 覆盖 `__getattribute__()` 可防止自动描述符调用
* `object.__getattribute__()` 和 `type.__getattribute__()` 对 `__get__()` 的调用方式不同。
* 数据描述符总是覆盖实例字典。
* 非数据描述符可能被实例字典覆盖。

`super()` 方法返回的对象也有一个自定义的 `__getattribute__()` 方法来调用描述符。`super(B, obj).m()` 被调用时，会先在 `obj.__class__.__mro__` 中查找与 `B` 紧邻的基类 `A`，然后返回 `A.__dict__['m'].__get__(obj, B)`。如果结果不是描述器，返回 `m`。如果 `m` 不再实例字典中，则回溯调用 `object.__getattribute__()` 继续查找。

上面的细节显示，`object`,`type` 和 `super()` 中描述符的实现是在 `__getattribute__()` 方法中。当它们从对象派生时，或者它们具有提供类似功能的元类时，类继承这个机制。同样，类可以通过覆盖 `__getattribute__()` 来关闭描述符调用。

## 描述符示例

以下代码创建一个类，其对象是数据描述符，它为每个 get 或 set 打印一条消息。覆盖 `__getattribute__()` 是替代方法，可以为每个属性执行此操作。但是，这个描述符对于监视几个选择的属性非常有用：

``` python
class RevealAccess(object):
    """A data descriptor that sets and returns values
       normally and prints a message logging their access.
    """

    def __init__(self, initval=None, name='var'):
        self.val = initval
        self.name = name

    def __get__(self, obj, objtype):
        print('Retrieving', self.name)
        return self.val

    def __set__(self, obj, val):
        print('Updating', self.name)
        self.val = val

>>> class MyClass(object):
...     x = RevealAccess(10, 'var "x"')
...     y = 5
...
>>> m = MyClass()
>>> m.x
Retrieving var "x"
10
>>> m.x = 20
Updating var "x"
>>> m.x
Retrieving var "x"
20
>>> m.y
5
```

协议很简单，并提供令人兴奋的可能性。几个用例非常常见，它们已被封装到单个函数调用中。属性，绑定方法，静态方法和类方法都基于描述符协议。

## 属性

调用 `property()` 是一种简洁的构建数据描述符的方法，可以在访问属性时触发函数调用。其签名是：

``` python
property(fget=None, fset=None, fdel=None, doc=None) -> property attribute
```

该文档显示了定义托管属性 `x` 的典型用法：

``` python
class C(object):
    def getx(self): return self.__x
    def setx(self, value): self.__x = value
    def delx(self): del self.__x
    x = property(getx, setx, delx, "I'm the 'x' property.")
```

为了了解 `property()` 是如何根据描述符协议实现的，下面是一个纯 Python 实现：

``` python
class Property(object):
    "Emulate PyProperty_Type() in Objects/descrobject.c"

    def __init__(self, fget=None, fset=None, fdel=None, doc=None):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel
        if doc is None and fget is not None:
            doc = fget.__doc__
        self.__doc__ = doc

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        if self.fget is None:
            raise AttributeError("unreadable attribute")
        return self.fget(obj)

    def __set__(self, obj, value):
        if self.fset is None:
            raise AttributeError("can't set attribute")
        self.fset(obj, value)

    def __delete__(self, obj):
        if self.fdel is None:
            raise AttributeError("can't delete attribute")
        self.fdel(obj)

    def getter(self, fget):
        return type(self)(fget, self.fset, self.fdel, self.__doc__)

    def setter(self, fset):
        return type(self)(self.fget, fset, self.fdel, self.__doc__)

    def deleter(self, fdel):
        return type(self)(self.fget, self.fset, fdel, self.__doc__)
```

当用户接口已经被授权访问属性后，需求发生一些变化，需要对属性做进一步处理才能返回给用户时，`property()` 就很有用了。

比如，电子表格类可能通过 `Cell('b10').value` 来访问单元格的值。对程序的后续改进需要在每次访问时重新计算单元格;然而，程序员不想影响直接访问属性的现有客户端代码。解决方案是在属性数据描述符中包装对 `value` 属性的访问：

``` python
class Cell(object):
    . . .
    def getvalue(self):
        "Recalculate the cell before returning value"
        self.recalc()
        return self._value
    value = property(getvalue)
```

## 函数和方法

Python 的面向对象特性建立在基于函数的环境之上。使用非数据描述符，两者可以无缝合并。类字典将方法存储为函数。在类定义中，方法使用 `def` 或 `lambda` 编写，这是创建函数的常用工具。方法只与常规函数不同，因为第一个参数是为对象实例保留的。按照 Python 约定，实例引用称为 `self`，但可以称为 `this` 或任何其他变量名称。

为了支持方法调用，函数在属性访问期间包含用于绑定方法的 `__get__()` 方法。这意味着所有函数都是非数据描述符，它们在从对象调用时返回绑定方法。在纯 Python 中，它的工作原理是这样的：

``` python
class Function(object):
    . . .
    def __get__(self, obj, objtype=None):
        "Simulate func_descr_get() in Objects/funcobject.c"
        if obj is None:
            return self
        return types.MethodType(self, obj)
```

运行解释器显示了函数描述符在实际中的工作方式：

``` python
>>> class D(object):
...     def f(self, x):
...         return x
...
>>> d = D()

# Access through the class dictionary does not invoke __get__.
# It just returns the underlying function object.
>>> D.__dict__['f']
<function D.f at 0x00C45070>

# Dotted access from a class calls __get__() which just returns
# the underlying function unchanged.
>>> D.f
<function D.f at 0x00C45070>

# The function has a __qualname__ attribute to support introspection
>>> D.f.__qualname__
'D.f'

# Dotted access from an instance calls __get__() which returns the
# function wrapped in a bound method object
>>> d.f
<bound method D.f of <__main__.D object at 0x00B18C90>>

# Internally, the bound method stores the underlying function,
# the bound instance, and the class of the bound instance.
>>> d.f.__func__
<function D.f at 0x1012e5ae8>
>>> d.f.__self__
<__main__.D object at 0x1012e1f98>
>>> d.f.__class__
<class 'method'>
```

## 静态方法和类方法

非数据描述符提供了用于将绑定函数的通常模式变化为方法的简单机制。

回顾一下，函数有一个 `__get__()` 方法，以便在作为属性访问时可以将它们转换为方法。非描述符描述符将 `obj.f(*args)` 调用转换为 `f(obj, *args)`。调用 `kclass.f(*args)` 会变成 `f(*args)`。

该图表总结了绑定及其两个最有用的变体：

|转换	|从对象调用	|从类调用
|:-----|:-----------|:---------
|函数	    |`f(obj, *args)`|	`f(*args)`
|静态方法	|`f(*args)`	|`f(*args)`
|类方法	|`f(type(obj), *args)`|	`f(klass, *args)`

静态方法会返回没有更改的底层函数。调用 `c.f` 或者是 `C.f` 等价交换于直接查找 `object.__getattribute__(c, "f")` 或者 `obj.__getattribute__(C, "f")`。因此，该函数从对象或者类中可以同等访问。

那些不需要 `self` 变量的适合写成静态方法。

例如，统计软件包可能包含实验数据的容器类。该类提供了用于计算平均数，中位数，以及其他描述性统计指标的方法。然而，可能存在部分功能，概念上相关但是不依赖于数据。例如，`erf(x)` 是在统计工作中出现的转换例程，但不直接依赖于特定的数据集。类或者是对象可以调用它: `s.erf(1.5) --> .9332` 或者 `Sample.erf(1.5) --> .9332`。

既然静态方法会将底层的函数原样返回，那么下面的代码就不会让人奇怪了。

``` python
>>> class E(object):
...     def f(x):
...         print(x)
...     f = staticmethod(f)
...
>>> print(E.f(3))
3
>>> print(E().f(3))
3
```

使用非数据描述符协议，Python 版本的 `staticmethod()` 如下:

``` python
class StaticMethod(object):
    "Emulate PyStaticMethod_Type() in Objects/funcobject.c"

    def __init__(self, f):
        self.f = f

    def __get__(self, obj, objtype=None):
        return self.f
```

与静态方法不同，在调用函数之前，类方法会将参数列表的类引用添加到参数列表中。这种格式对于调用者是对象还是类是相同的：

``` python
>>> class E(object):
...     def f(klass, x):
...         return klass.__name__, x
...     f = classmethod(f)
...
>>> print(E.f(3))
('E', 3)
>>> print(E().f(3))
('E', 3)
```

在函数只需要关心类引用儿不需要关心任何底层数据时，这个特性是很有用的。类方法的一个用途是创建构造函数。在 Python2.3 中，类方法 `dict.fromkeys()` 会依据一个 key 列表来创建一个新的字典。等价的 Python 实现如下。

``` python
class Dict(object):
    . . .
    def fromkeys(klass, iterable, value=None):
        "Emulate dict_fromkeys() in Objects/dictobject.c"
        d = klass()
        for key in iterable:
            d[key] = value
        return d
    fromkeys = classmethod(fromkeys)
```

现在可以像这样构建一个新的唯一键的字典：

``` python
>>> Dict.fromkeys('abracadabra')
{'a': None, 'r': None, 'b': None, 'c': None, 'd': None}
```

使用非数据描述符协议，`classmethod()` 的纯 Python 版本看起来像这样：

``` python
class ClassMethod(object):
    "Emulate PyClassMethod_Type() in Objects/funcobject.c"

    def __init__(self, f):
        self.f = f

    def __get__(self, obj, klass=None):
        if klass is None:
            klass = type(obj)
        def newfunc(*args):
            return self.f(klass, *args)
        return newfunc
```