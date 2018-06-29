# 元类的使用

先看一个例子

``` python
class MyMetaclass(type):

    def __new__(cls, name, bases, attr_dict):
        print('__new__  name  ==', name)
        print('__new__  bases ==', bases)
        for key, attr in attr_dict.items():
            print('{:^20} = {}'.format(key, attr))
        return super().__new__(cls, name, bases, attr_dict)

    def __init__(self, name, bases, attr_dict):
        print('__init__ name  ==', name)
        print('__init__ bases ==', bases)
        for key, attr in attr_dict.items():
            print('{:^20} = {}'.format(key, attr))

class MyClass(object, metaclass = MyMetaclass):
    aaaaaaaa = '测试代码'

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def test1(self):
        pass

    def test2(self):
        pass

```

执行这段代码结果如下

```
__new__  name  == MyClass
__new__  bases == (<class 'object'>,)
     __module__      = __main__
    __qualname__     = MyClass
      aaaaaaaa       = 测试代码
      __init__       = <function MyClass.__init__ at 0x7ff52744e488>
       test1         = <function MyClass.test1 at 0x7ff52744e510>
       test2         = <function MyClass.test2 at 0x7ff52744e598>
__init__ name  == MyClass
__init__ bases == (<class 'object'>,)
     __module__      = __main__
    __qualname__     = MyClass
      aaaaaaaa       = 测试代码
      __init__       = <function MyClass.__init__ at 0x7ff52744e488>
       test1         = <function MyClass.test1 at 0x7ff52744e510>
       test2         = <function MyClass.test2 at 0x7ff52744e598>
```

可以发现元类的一些特性

`__new__()` 方法有三个额外参数： `name`, `bases`, `attr_dict`

* `name`：是其实现类的类名
* `bases`：是其实现类继承的类的元组
* `attr_dict`：是一个 `dict`，其中包含了实现类的所有类属性（方法也算是类属性）

因此，我们可以在运行时对类进行动态修改

!> `__new__()` 必须返回一个对象，一般是「实现类」对象，这个对象就是后面 `__init__()` 中的 `self`

这些参数都会传递给元类的 `__init__()` 方法

看个例子，我们给使用了元类的类自动添加一个函数

``` python
class MyMetaclass(type):

    def __new__(cls, name, bases, attr_dict):

        def func_a(self):
            print('我来自元类')
        attr_dict['func_a'] = func_a
        return super().__new__(cls, name, bases, attr_dict)


class MyClass(metaclass = MyMetaclass):
    pass

m = MyClass()
m.func_a()

# 我来自元类
```