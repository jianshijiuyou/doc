> [内容都来自崔庆才的博客和书](https://cuiqingcai.com)

# 连接 MongoDB

两种方式

``` python
from pymongo import MongoClient

#client = MongoClient(host='localhost', '27017')
client = MongoClient('mongodb://localhost:27017/')
```

# 指定数据库

两种方式

``` python
# db = client.test
db = client['test']
```

# 指定集合

``` python
# collection = db.students
collection = db['students']
```

# 插入数据

## insert_one

插入单条用 insert_one

``` python
from pymongo import MongoClient

student = {
    'id': '534533',
    'name': 'jack',
    'age': 19,
    'gender': 'male'
}

client = MongoClient('mongodb://localhost:27017/')
db = client['test']
collection = db['students']
result = collection.insert_one(student)
print(result)

# <pymongo.results.InsertOneResult object at 0x7f2f38e78748>
```

## insert_many

一次插入多条用 insert_many

``` python
student1 = {
    'id': '123233',
    'name': '张三',
    'age': 32,
    'gender': 'male'
}

student2 = {
    'id': '342435',
    'name': '里斯',
    'age': 44,
    'gender': 'male'
}

result = collection.insert_many([student1, student2])
print(result.inserted_ids)

# [ObjectId('5b33a8105f627d3ddbf497ec'), ObjectId('5b33a8105f627d3ddbf497ed')]
```

# 查询

## find_one

返回单个结果

``` python
db = client['test']
collection = db['students']
result = collection.find_one({'name': 'jack'})
print(type(result))
print(result)

# <class 'dict'>
# {'_id': ObjectId('5b33a70c5f627d3d987668e9'), 'id': '534533', 'name': 'jack', 'age': 19, 'gender': 'male'}
```

根据 ObjectId 查询

``` python
from bson.objectid import ObjectId
result = collection.find_one({'_id': ObjectId('5b33a8105f627d3ddbf497ec')})
print(result)
```

## find

返回生成器对象，用于查询多个结果

``` python
results = collection.find({'gender': 'male'})
for student in results:
    print(student)
```

## 条件查询

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/images/20180627232537.png)

查询年龄大于 30 的学生

``` python
results = collection.find({'age': {'$gt': 30}})
```

更加复杂的查询方式

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/images/20180627233413.png)

# 计数

count()

``` python
count = collection.find().count()
```

# 排序

sort()

``` python
import pymongo
result = collection.find().sort('age', pymongo.DESCENDING) # pymongo.ASCENDING
```

# 偏移

``` python
result = collection.find().sort('age', pymongo.ASCENDING).skip(1).limit(1)
```

skip 跳过多少条记录

limit 指定获取记录条数

!> 数据量太大时千万别用这种方式取数据，应该像下面这样

``` python
result = collection.find({'_id': {'$gt': ObjectId('5b33a8105f627d3ddbf497ec')}})
```

# 更新

update()

``` python
condition = {'name': '张三'}
student = collection.find_one(condition)
student['age'] = 13
result = collection.update(condition, student)
print(result)

# {'n': 1, 'nModified': 1, 'ok': 1.0, 'updatedExisting': True}

```

或者用 `$set`

``` python
result = collection.update(condition, {'$set': student})
```

!> 这样可以只更新 `student` 字典内存在的字段。如果原先还有其他字段，则不会更新，也不会删除。而如果不用 `$set` 的话，则会把之前的数据全部用 `student` 字典替换；如果原本存在其他字段，则会被删除。

!> 不推荐时用 update 方法，应该使用 update_one 和 update_many 方法

``` python
result = collection.update_one(condition, {'$set': student})
print(result.matched_count, result.modified_count)

# 1 1 
# 匹配条数 影响条数
```

将年龄大于 16 的人岁数加一

``` python
result = collection.update_many({'age': {'$gt': 16}}, {'$inc': {'age': 1}})
print(result.matched_count, result.modified_count)
```

## 更新插入

存在即更新，不存在则插入

``` python
collection.update({'nickname': 'xxx'}, {'age': 18}, True)
```

第三个参数为 True 则实现上述逻辑

# 删除

``` python
result = collection.remove({'name': 'jack'})
print(result)

# {'n': 1, 'ok': 1.0}
```

推荐使用 delete_one 和 delete_many

``` python
result = collection.delete_one({'name': 'jack1'})
print(result.deleted_count)
result = collection.delete_many({'age': {'$gt': 60}})
print(result.deleted_count)
```

# 其他操作

find_one_and_delete() 查找后删除

find_one_and_replace() 查找后替换

find_one_and_update() 查找后更新