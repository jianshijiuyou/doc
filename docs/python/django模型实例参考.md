# 模型实例参考

## 创建模型

`class Model(**kwargs)` [[source]](https://docs.djangoproject.com/zh-hans/2.0/_modules/django/db/models/base/#Model)

关键字参数是您在模型上定义的字段的名称。请注意，实例化模型不会触及数据库;为此，您需要 `save()`。

> 如果要在模型上自定义初始化方法，请使用以下方式之一。<br> <br>
> 1. 在模型类上添加一个 `classmethod`：
>    ``` python
>    from django.db import models
>
>    class Book(models.Model):
>        title = models.CharField(max_length=100)
>
>        @classmethod
>        def create(cls, title):
>            book = cls(title=title)
>            # do something with the book
>            return book
>
>    book = Book.create("Pride and Prejudice")
>    ```
> 2. 在自定义管理器中添加一个方法（通常是首选）：
>    ``` python
>    class BookManager(models.Manager):
>        def create_book(self, title):
>            book = self.create(title=title)
>            # do something with the book
>            return book
>    
>    class Book(models.Model):
>        title = models.CharField(max_length=100)
>    
>        objects = BookManager()
>    
>    book = Book.objects.create_book("Pride and Prejudice")
>    ```

## 刷新数据库中的对象

如果从模型实例中删除一个字段，则再次访问该字段会重新从数据库中载入值：

``` python
>>> obj = MyModel.objects.first()
>>> del obj.field
>>> obj.field  # Loads the field from the database
```

如果您需要从数据库重新加载模型的值，则可以使用该 `refresh_from_db()` 方法。

## 保存对象

要将对象保存回数据库，请调用 `save()`：

### 自动递增主键

如果一个模型有一个 `AutoField` - 一个自动递增的主键 - 那么当您第一次调用 `save()` 时，该自动递增的值将被计算并保存为对象的一个​​属性：

``` python
>>> b2 = Blog(name='Cheddar Talk', tagline='Thoughts on cheese.')
>>> b2.id     # Returns None, because b doesn't have an ID yet.
>>> b2.save()
>>> b2.id     # Returns the ID of your new object.
```

在调用 `save()` 之前，没有办法知道 id 的值是什么，因为该值是由数据库而不是由 Django 计算的。

为方便起见，除非您在模型中的字段上明确指定 `primary_key=True`，否则每个模型默认都有一个名为 `id` 的 `AutoField`。

#### pk 属性

`model.pk`

无论您是自己定义主键字段还是让 Django 为您提供主键字段，每个模型都会有一个名为 `pk` 的属性。它表现得像模型上的普通属性，但实际上是模型的主键字段的别名。您可以像读取任何其他属性一样读取和设置此值，并且它将更新模型中的正确字段。

#### 显式指定自动主键值

如果模型具有 `AutoField`，但您希望在保存时明确定义新对象的 `ID`，则只需在保存前明确定义它，而不是依赖 `ID` 的自动分配：

``` python
>>> b3 = Blog(id=3, name='Cheddar Talk', tagline='Thoughts on cheese.')
>>> b3.id     # Returns 3.
>>> b3.save()
>>> b3.id     # Returns 3.
```

如果您手动分配自动主键值，请确保不要使用已有的主键值！如果您使用数据库中已存在的显式主键值创建新对象，Django会假定您正在更改现有记录而不是创建新记录。

### 当你调用 `save()` 时会发生什么？


## 删除对象