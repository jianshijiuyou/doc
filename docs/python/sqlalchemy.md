> [官方指南](http://docs.sqlalchemy.org/en/latest/orm/tutorial.html)  
> [参考链接](http://www.cnblogs.com/iwangzc/p/4112078.html)

# 基本用法

## 连接

``` python
from sqlalchemy import create_engine
# engine = create_engine('mysql+pymysql://username:password@ip:port/database')
engine = create_engine('sqlite:///:memory:',echo=True)
```

`echo` 参数为 `True` 时，会显示每条执行的 SQL 语句，可以关闭。`create_engine()` 返回一个 `Engine` 的实例，它表示通过数据库语法处理细节的核心接口，在这种情况下，数据库语法将会被解释称 Python 的类方法。

> [MySql 的各种方言支持详情](http://docs.sqlalchemy.org/en/latest/dialects/mysql.html)

## 声明映射类

第一步，创建一个 `declarative_base` 对象

``` python
from sqlalchemy.ext.declarative import declarative_base
Base = declarative_base()
```

然后就可以创建映射类了

``` python
>>> from sqlalchemy import Column, Integer, String
>>> class User(Base):
...     __tablename__ = 'users'
...
...     id = Column(Integer, primary_key=True)
...     name = Column(String)
...     fullname = Column(String)
...     password = Column(String)
...
...     def __repr__(self):
...        return "<User(name='%s', fullname='%s', password='%s')>" % (
...                             self.name, self.fullname, self.password)
```

!> 用 `Declarative` 构造的一个类至少需要一个 `__tablename__` 属性，一个主键行。

## 创建数据库表

如果数据库表已经存在，则可以不执行

``` python
>>> Base.metadata.create_all(engine)
SELECT ...
PRAGMA table_info("users")
()
CREATE TABLE users (
    id INTEGER NOT NULL, name VARCHAR,
    fullname VARCHAR,
    password VARCHAR,
    PRIMARY KEY (id)
)
()
COMMIT
```

## 创建映射类实例

完成映射后，现在让我们创建并检查一个 `User` 对象：

``` python
>>> ed_user = User(name='ed', fullname='Ed Jones', password='edspassword')
>>> ed_user.name
'ed'
>>> ed_user.password
'edspassword'
>>> str(ed_user.id)
'None'
```

## 创建会话

现在我们已经准备毫和数据库开始会话了。ORM 通过 Session 与数据库建立连接的。当应用第一次载入时，我们定义一个 Session 类（声明 `create_engine()` 的同时），这个 Session 类为新的 Session 对象提供工厂服务。

``` python
from sqlalchemy.orm import sessionmaker
Session = sessionmaker(bind=engine)
# Session.configure(bind=engine) 
session = Session()
```

## 添加和更新对象

``` python
ed_user = User(name='ed', fullname='Ed Jones', password='edspassword')
session.add(ed_user)
```

可以使用 `add_all()` 一次添加多个

``` python
>>> session.add_all([
...     User(name='wendy', fullname='Wendy Williams', password='foobar'),
...     User(name='mary', fullname='Mary Contrary', password='xxg527'),
...     User(name='fred', fullname='Fred Flinstone', password='blah')])
```

修改数据

``` python
ed_user.password = 'f8s7ccs'
```

查看被修改的数据

``` python
>>> session.dirty
IdentitySet([<User(name='ed', fullname='Ed Jones', password='f8s7ccs')>])
```

查看新添加的数据

``` python
>>> session.new  
IdentitySet([<User(name='wendy', fullname='Wendy Williams', password='foobar')>,
<User(name='mary', fullname='Mary Contrary', password='xxg527')>,
<User(name='fred', fullname='Fred Flinstone', password='blah')>])
```

将数据提交到数据库

``` python
>>> session.commit()
```

## 回滚

``` python
>>> ed_user.name = 'Edwardo'

>>> fake_user = User(name='fakeuser', fullname='Invalid', password='12345')
>>> session.add(fake_user)

>>> session.query(User).filter(User.name.in_(['Edwardo', 'fakeuser'])).all()
[<User(name='Edwardo', fullname='Ed Jones', password='f8s7ccs')>, <User(name='fakeuser', fullname='Invalid', password='12345')>]

>>> session.rollback()

>>> ed_user.name
u'ed'

>>> fake_user in session
False
```



# 查询

通过 `Session` 的 `query()` 方法创建一个查询对象。这个函数的参数数量是可变的，参数可以是任何类或者是类的描述的集合。

``` python
for instance in session.query(User).order_by(User.id):
    print(instance.name, instance.fullname)
```

多个类的实体或者是基于列的实体表达都可以作为 `query()` 函数的参数

``` python
for name, fullname in session.query(User.name,User.fullname):
    print(name, fullname)
```

Query 返回的元组被命名为 `KeyedTuple` 类的实例元组。并且可以把它当成一个普通的 Python 数据类操作。元组的名字就相当于属性的属性名，类的类名一样。

``` python
for row in session.query(User, User.name).all():
    print(row.User, row.name)
    # <User(name='ed',fullname='Ed Jones', password='f8s7ccs')> ed
```

## label

``` python
for row in session.query(User.name.label('name_label')).all():
    print(row.name_label)
```

## aliased

类的别名

``` python
from sqlalchemy.orm import aliased
user_alias = aliased(User, name='user_alias')

for row in session.query(user_alias,user_alias.name).all():
    print row.user_alias
```

下面我们 join `Address` 实体两次，找到同时拥有两个不同 `email` 的用户：

``` python
from sqlalchemy.ormimport aliased

adalias1 = aliased(Address)
adalias2 = aliased(Address)

for username, email1, email2 in\
session.query(User.name,adalias1.email_address,adalias2.email_address).\
join(adalias1, User.addresses).\
join(adalias2, User.addresses).\
filter(adalias1.email_address=='jack@google.com').\
filter(adalias2.email_address=='j25@yahoo.com'):
    print(username, email1, email2)
    # jack jack@google.com j25@yahoo.com
```

## 用切片分页

``` python
for u in session.query(User).order_by(User.id)[1:3]:
# 只查询第二条和第三条数据
```

## filter

``` python
query.filter(User.name == 'ed') #equals
query.filter(User.name != 'ed') #not equals
query.filter(User.name.like('%ed%')) #LIKE
uery.filter(User.name.in_(['ed','wendy', 'jack'])) #IN
query.filter(User.name.in_(session.query(User.name).filter(User.name.like('%ed%'))#IN
query.filter(~User.name.in_(['ed','wendy', 'jack']))#not IN
query.filter(User.name == None)#is None
query.filter(User.name != None)#not None

from sqlalchemy import and_

query.filter(and_(User.name =='ed',User.fullname =='Ed Jones')) # and
query.filter(User.name == 'ed',User.fullname =='Ed Jones') # and
query.filter(User.name == 'ed').filter(User.fullname == 'Ed Jones')# and

from sqlalchemy import or_

query.filter(or_(User.name =='ed', User.name =='wendy')) #or
query.filter(User.name.match('wendy')) #match
```

## all

`all()` 返回一个列表：可以进行 Python 列表的操作。

``` python
query = session.query(User).filter(User.name.like('%ed')).order_by(User.id)
query.all()

# [<User(name='ed',fullname='EdJones', password='f8s7ccs')>,<User(name='fred',
# fullname='FredFlinstone', password='blah')>]
```

## first

返回查询到的第一条记录

``` python
query.first()
```

## one

结果只能为一条数据，为空则抛出 `NoResultFound`，多条则抛出 `MultipleResultsFound`

``` python
# from sqlalchemy.orm.exc import MultipleResultsFound, NoResultFound
query.one()
```

## scalar

在 `one()` 成功的基础上返回行的第一列。

``` python
query = session.query(User.id).filter(User.name == 'ed')
query.scalar()
```

## 字符串 SQL

字符串能使 Query 更加灵活，通过 `text()` 构造指定字符串的使用，这种方法可以用在很多方法中，像 `filter()` 和 `order_by()`。

``` python
from sqlalchemy import text
for user in session.query(User).filter(text("id<224")).order_by(text("id")).all()
```

绑定参数可以指定字符串，用 `params()` 方法指定数值。

``` python
session.query(User).filter(text("id<:value and name=:name")).\
params(value=224, name='fred').order_by(User.id).one()
```

如果要用一个完整的 SQL 语句，可以使用 `from_statement()`。

``` python
ession.query(User).from_statement(text("SELECT* FROM users where name=:name")).\
			params(name='ed').all()
```

也可以用 `from_statement()` 获取完整的 ”raw”，用字符名确定希望被查询的特定列:

``` python
session.query("id","name", "thenumber12").\
            from_statement(text("SELECT id, name, 12 as thenumber12 FROM users where name=:name")).\
            params(name='ed').all()

# [(1,u'ed', 12)]
```

## count

`count()` 用来统计查询结果的数量。

``` python
session.query(User).filter(User.name.like('%ed')).count()
```

`func.count()` 方法比 `count()` 更高级一点

``` python
from sqlalchemy import func
session.query(func.count(User.name),User.name).group_by(User.name).all()
# [(1,u'ed'), (1,u'fred'), (1,u'mary'), (1,u'wendy')]
```

`SELECT count(*) FROM table` 可以这么写：

``` python
session.query(func.count('*')).select_from(User).scalar()
```

如果我们明确表达计数是根据 User 表的主键的话，可以省略 `select_from(User)`:

``` python
session.query(func.count(User.id)).scalar()
```

## join

``` python
session.query(User).join(Address).\
filter(Address.email_address=='jack@google.com').\
all() 
# [<User(name='jack',fullname='JackBean', password='gjffdd')>]
```

之所以 `Query.join()` 知道怎么 `join` 两张表是因为它们之间只有一个外键。

如果两张表中没有外键或者有一个以上的外键，当下列几种形式使用的时候，`Query.join()` 可以表现的更好：

``` python
query.join(Address,User.id==Address.user_id)# 明确的条件
query.join(User.addresses)# 指定从左到右的关系
query.join(Address,User.addresses)    #同样，有明确的目标
query.join('addresses') # 同样，使用字符串
```

`outerjoin()` 和 `join()` 用法相同

``` python
query.outerjoin(User.addresses)# LEFT OUTER JOIN
```

# 外键约束

## 创建外键

假设我们系统的用户可以存储任意数量的 `email` 地址。我们需要定义一个新表 `Address` 与 `User` 相关联。

``` python
>>> from sqlalchemy import ForeignKey
>>> from sqlalchemy.orm import relationship

>>> class Address(Base):
...     __tablename__ = 'addresses'
...     id = Column(Integer, primary_key=True)
...     email_address = Column(String, nullable=False)
...     user_id = Column(Integer, ForeignKey('users.id'))
...
...     user = relationship("User", back_populates="addresses")
...
...     def __repr__(self):
...         return "<Address(email_address='%s')>" % self.email_address

>>> User.addresses = relationship(
...     "Address", order_by=Address.id, back_populates="user")
```

`relationship()`

这个函数告诉 ORM，`Address` 类应该和 `User` 类连接起来，通过使用 `address.user`。`relationship()` 使用外键明确这两张表的关系。决定`Adderess.user` 属性是多对一的。`back_populates` 提供表达反向关系的引用属性：`relationship()` 对象的集合被`User.addresses` 引用。多对一的反向关系总是一对多。

这两个互补关系：`Address.user`和 `User.addresses` 被称为双向关系。

!> `relationship.back_populates` 参数是 `relationship.backref` 的较新版本。

下面是一个在 `User` 类中创建双向联系的例子：

``` python
class User(Base):
    addresses = relationship("Address", order_by="Address.id", back_populates="user")
```

## 操作外键

``` python
jack = User(name='jack', fullname='Jack Bean', password='gjffdd')
jack.addresses

# []
```

现在可以直接在 `User` 对象中添加 `Address` 对象。

``` python
jack.addresses = [Address(email_address='jack@google.com'),Address(email_address='j25@yahoo.com')]

# 要保存到数据库还是要提交 jack 的
>>> jack.addresses[1]
<Address(email_address='j25@yahoo.com')>
>>> jack.addresses[1].user
<User(name='jack', fullname='Jack Bean', password='gjffdd')>
```

# 删除

``` python
>>> session.delete(jack)
SQL>>> session.query(User).filter_by(name='jack').count()
0
```

用户删除了，但是关联的地址还在

``` python
>>> session.query(Address).filter(
...     Address.email_address.in_(['jack@google.com', 'j25@yahoo.com'])
...  ).count()
2
```

SQLAlchemy 默认是将相关字段设置为 `None`，而不是级联删除，这需要显示告诉它

``` python
>>> class User(Base):
...     __tablename__ = 'users'
...
...     id = Column(Integer, primary_key=True)
...     name = Column(String)
...     fullname = Column(String)
...     password = Column(String)
...
...     addresses = relationship("Address", back_populates='user',
...                     cascade="all, delete, delete-orphan")
...
...     def __repr__(self):
...        return "<User(name='%s', fullname='%s', password='%s')>" % (
...                                self.name, self.fullname, self.password)
```

``` python
>>> class Address(Base):
...     __tablename__ = 'addresses'
...     id = Column(Integer, primary_key=True)
...     email_address = Column(String, nullable=False)
...     user_id = Column(Integer, ForeignKey('users.id'))
...     user = relationship("User", back_populates="addresses")
...
...     def __repr__(self):
...         return "<Address(email_address='%s')>" % self.email_address
```

# 第三方库

[sqlalchemy-mixins](https://github.com/absent1706/sqlalchemy-mixins)

像 Django ORM 一样操作查询

[sqlalchemy-repr](https://github.com/manicmaniac/sqlalchemy-repr)

自动生成一个 SQLAlchemy 模型的漂亮 `repr`。


# 其他

## 打印 sql 语句

https://docs.sqlalchemy.org/en/latest/core/engines.html#dbengine-logging

``` python
import logging

logging.basicConfig()
logging.getLogger('sqlalchemy.engine').setLevel(logging.INFO)
```