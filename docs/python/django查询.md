# 查询操作

一旦创建了数据模型，Django 就会自动为您提供一个数据库抽象 API，使您可以创建，检索，更新和删除对象。本文档介绍了如何使用此 API。


在本指南（和参考文献）中，我们将参考以下模型，它们构成了一个 Weblog 应用程序：

``` python
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def __str__(self):
        return self.name

class Author(models.Model):
    name = models.CharField(max_length=200)
    email = models.EmailField()

    def __str__(self):
        return self.name

class Entry(models.Model):
    blog = models.ForeignKey(Blog, on_delete=models.CASCADE)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField()
    authors = models.ManyToManyField(Author)
    n_comments = models.IntegerField()
    n_pingbacks = models.IntegerField()
    rating = models.IntegerField()

    def __str__(self):
        return self.headline
```

## 创建对象

为了在 Python 对象中表示数据库表数据，Django 使用直观的系统：模型类表示数据库表，该类的实例表示数据库表中的特定记录。

要创建一个对象，请使用模型类的关键字参数对其进行实例化，然后调用 `save()` 以将其保存到数据库。

假设模型存在于文件 `mysite/blog/models.py` 中，下面是一个例子：

``` python
>>> from blog.models import Blog
>>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
>>> b.save()
```

幕后执行 SQL 语句 INSERT 。 在你没有调用 `save()` 方法之前， Django 不会将数据保存到数据库

`save()` 方法没有返回值。

## 保存对对象的更改

要保存对已存在于数据库中的对象的更改，请使用 `save()`。

给定一个已经保存到数据库的 `Blog` 实例 `b5`，这个例子改变它的名字并更新数据库中的记录：

``` python
>>> b5.name = 'New name'
>>> b5.save()
```

幕后执行 SQL 语句 UPDATE 。 在你没有调用 `save()` 方法之前， Django 不会将数据保存到数据库

### 保存 ForeignKey 和 ManyToManyField 字段

更新 ForeignKey 字段的方式与保存普通字段的方式完全相同 -- 只需将正确类型的对象分配给相关字段即可。

``` python
>>> from blog.models import Blog, Entry
>>> entry = Entry.objects.get(pk=1)
>>> cheese_blog = Blog.objects.get(name="Cheddar Talk")
>>> entry.blog = cheese_blog
>>> entry.save()
```

还可以通过 `add()` 方法：

``` python
>>> from blog.models import Author
>>> joe = Author.objects.create(name="Joe")
>>> entry.authors.add(joe)
```
要一次添加多个记录：

``` python
>>> john = Author.objects.create(name="John")
>>> paul = Author.objects.create(name="Paul")
>>> george = Author.objects.create(name="George")
>>> ringo = Author.objects.create(name="Ringo")
>>> entry.authors.add(john, paul, george, ringo)
```

## 检索对象

要从数据库中检索对象，请在您的模型类上通过 `Manager` 构建一个 `QuerySet`。

`QuerySet` 表示数据库中的对象集合。它可以有零个，一个或多个过滤器。过滤器根据给定的参数缩小查询结果的范围。在 SQL 术语中，`QuerySet` 等同于 SELECT 语句，过滤器是限制性子句，如 WHERE 或 LIMIT。

您可以使用模型的 `Manager` 获取 `QuerySet`。每个模型至少有一个 `Manager`，默认情况下是 `objects`。通过模型类直接访问它，如下所示：

``` python
>>> Blog.objects
<django.db.models.manager.Manager object at ...>
>>> b = Blog(name='Foo', tagline='Bar')
>>> b.objects
Traceback:
    ...
AttributeError: "Manager isn't accessible via Blog instances."
```
!> Managers 只能通过模型​​类访问，而不能从模型实例访问。

`Manager` 是模型的 `QuerySet` 的主要来源。例如，`Blog.objects.all()` 返回包含数据库中所有 `Blog` 对象的 `QuerySet`。

### 检索所有对象

从表中检索对象的最简单方法是获取所有对象。为此，请使用 `Manager` 上的 `all()` 方法：

``` python
>>> all_entries = Entry.objects.all()
```

`all()` 方法返回数据库中所有对象的 `QuerySet`。

### 使用过滤器检索特定的对象

`filter(**kwargs)`

返回包含匹配给定查找参数的对象的新 `QuerySet`。

`exclude(**kwargs)`

返回包含与给定查找参数不匹配的对象的新 `QuerySet`。

例如，要获取从 2006 年开始的博客条目的 `QuerySet`，请使用 `filter()`，如下所示：

``` python
Entry.objects.filter(pub_date__year=2006)
```

使用默认 manager 类，它与以下内容相同：

``` python
Entry.objects.all().filter(pub_date__year=2006)
```

#### 链式过滤器

改进 `QuerySet` 的结果本身就是一个 `QuerySet`，因此可以将改进链接在一起。例如：

``` python
>>> Entry.objects.filter(
...     headline__startswith='What'
... ).exclude(
...     pub_date__gte=datetime.date.today()
... ).filter(
...     pub_date__gte=datetime.date(2005, 1, 30)
... )
```

#### 过滤后的 `QuerySet` 是独一无二的

每次您完善 `QuerySet` 时，都会获得全新的 `QuerySet`，它绝不会绑定到以前的 `QuerySet`。每个优化都会创建一个独立且不同的 `QuerySet` ，可以存储，使用和重用。

举例：

``` python
>>> q1 = Entry.objects.filter(headline__startswith="What")
>>> q2 = q1.exclude(pub_date__gte=datetime.date.today())
>>> q3 = q1.filter(pub_date__gte=datetime.date.today())
```

#### QuerySet 会延迟查询(lazy)

`QuerySets` 是懒惰的 -- 创建 `QuerySet` 的行为不涉及任何数据库活动。您可以将过滤器堆叠在一起，并且在使用 `QuerySet` 之前，Django 不会真正运行查询。看看这个例子：

``` python
>>> q = Entry.objects.filter(headline__startswith="What")
>>> q = q.filter(pub_date__lte=datetime.date.today())
>>> q = q.exclude(body_text__icontains="food")
>>> print(q)
```

看起来像是操作了三次数据库，实际上只有在执行 `print(p)` 时才会真正执行一次数据库查询操作。

### 用 get() 检索单个对象

`filter()` 总是返回一个 `QuerySet`，即使匹配的对象只有一个。

如果你知道只有一个对象和查询匹配，则可以直接使用 `get()` 方法返回单个对象：

``` python
>>> one_entry = Entry.objects.get(pk=1)
```

!> 请注意，`get()` 方法如果没有查找到对象，将引发 `DoesNotExist` 异常。查询到多个对象则会引发 `MultipleObjectReturned` 异常，这两个异常都是模型类的一个属性。

### 截取 QuerySet

可以使用 Python 中的切片语法对 `QuerySet` 的数量进行限制。这相当于 SQL LIMIT 和 OFFSET 子句。

例如，返回前 5 个对象（`LIMIT 5`）。

``` python
>>> Entry.objects.all()[:5]
```

返回第六到第十个对象（`OFFSET 5 LIMIT 5`）。

``` python
>>> Entry.objects.all()[5:10]
```


### 字段查找

### 查找跨越关系

#### 跨越多值关系

### 过滤器可以引用模型上的字段

### 在pk查找快捷

### 在LIKE报表中转义百分号和下划线

### 缓存和QuerySets

#### 当QuerySets没有被缓存时

## 使用Q对象进行复杂查找

## 比较对象

## 删除对象

## 复制模型实例

## 一次更新多个对象

## 相关对象

### 一对多的关系

#### 前锋
#### 关系“落后”
#### 使用自定义反向管理器
#### 处理相关对象的其他方法

### 多对多的关系

### 一对一的关系

### 落后的关系如何可能？

### 查询相关对象

## 回落到原始SQL