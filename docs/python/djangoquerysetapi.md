# QuerySet API

`class QuerySet(model=None, query=None, using=None)` [[source]](https://docs.djangoproject.com/zh-hans/2.0/_modules/django/db/models/query/#QuerySet)

QuerySet 类具有两个公有属性用于内省：

`ordered`

如果 `QuerySet` 是排好序的则为 `Tru`e —— 例如有一个 `order_by()` 子句或者模型有默认的排序。 否则为 `False` 。

`db`

如果现在执行，则返回将使用的数据库。

## 返回新 QuerySet 的方法

Django 提供了一系列 的 `QuerySet` 筛选方法，用于改变 `QuerySet` 返回的结果类型或者 SQL 查询执行的方式。

### filter()

`filter(**kwargs)`

返回一个新的QuerySet，它包含满足查询参数的对象。

查找的参数（**kwargs）应该满足下文字段查找中的格式。 在底层的 SQL 语句中，多个参数通过 `AND` 连接。

如果你需要执行更复杂的查询（例如 `OR` 语句），你可以使用 `Q` 对象。

### exclude()

`exclude(**kwargs)`

返回一个新的 `QuerySet`，它包含不满足给定的查找参数的对象。

查找的参数（**kwargs）应该满足下文字段查找中的格式。 在底层的 `SQL` 语句中，多个参数通过 `AND` 连接，然后所有的内容放入 `NOT()` 中。

下面的示例排除所有 `pub_date` 晚于 `2005-1-3` 且 `headline` 为 `"Hello"` 的记录：

``` python
Entry.objects.exclude(pub_date__gt=datetime.date(2005, 1, 3), headline='Hello')
```

它等同于 `SQL` 语句：

``` python
SELECT ...
WHERE NOT (pub_date > '2005-1-3' AND headline = 'Hello')
```

下面的示例排除所有 `pub_date` 晚于 `2005-1-3` 或者 `headline` 为 `"Hello"` 的记录：

``` python
Entry.objects.exclude(pub_date__gt=datetime.date(2005, 1, 3)).exclude(headline='Hello')
```

它等同于 `SQL` 语句：

``` python
SELECT ...
WHERE NOT pub_date > '2005-1-3'
AND NOT headline = 'Hello'
```

### annotate()
### order_by()
### reverse()
### distinct()
### values()
### values_list()
### dates()
### datetimes()
### none()
### all()
### union()
### intersection()
### difference()
### select_related()
### prefetch_related()
### extra()
### defer()
### only()
### using()
### select_for_update()
### raw()

## 不返回 QuerySet 的方法

### get()
### create()
### get_or_create()
### update_or_create()
### bulk_create()
### count()
### in_bulk()
### iterator()
### latest()
### earliest()
### first()
### last()
### aggregate()
### exists()
### update()
### delete()
### as_manager()

## 字段查找

### exact
### iexact
### contains
### icontains
### in
### gt
### gte
### lt
### lte
### startswith
### istartswith
### endswith
### iendswith
### range
### date
### year
### month
### day
### week
### week_day
### quarter
### time
### hour
### minute
### second
### isnull
### regex
### iregex

## 聚合函数

### expression
### output_field
### filter
### **extra
### Avg
### Count
### Max
### Min
### StdDev
### Sum
### Variance

# 查询相关工具

## Q() 对象

## Prefetch() 对象

## prefetch_related_objects()

## FilteredRelation() 对象