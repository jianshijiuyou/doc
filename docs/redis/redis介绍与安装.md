# 安装

在 ubuntu 下安装 Redis

安装编译工具 gcc

```
sudo apt update
sudo apt install make gcc
```

下载编译安装

```
wget http://download.redis.io/releases/redis-4.0.9.tar.gz
tar -xzvf redis-4.0.9.tar.gz
cd redis-4.0.9/
make
cd src
make all
cd ..
sudo make install
cd src
sudo make install
```

启动 Redis 服务器

```
sudo redis-server redis.conf
```

安装 Python 操作 Redis 的库

```
sudo pip install redis hiredis
```
!> hiredis 包是一个 C 库，它可以提高 Python 的 Redis 客户端库的速度。

测试

``` python
>>> import redis
>>> conn = redis.Redis()
>>> conn.set('hello', 'world')
True
>>> conn.get('hello')
'world'
```

或者

```
☁  ~  redis-cli
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> get hello
"world"
127.0.0.1:6379>
```

# 概念

## 分片

分片是一种将数据划分为多个部分的方法，对数据的划分可以基于键包含的 ID，基于键的散列值，或者基于以上两者的某种组合。通过对数据进行分片，用户可以将数据存储到多台机器里面，也可以从多台机器里面获取数据，这种方法在解决某些问题时可以获得线性的性能提升。

## 常见数据库的区别

| 名称 | 类型 | 数据存储选项 | 查询类型 | 附加功能 | 
|:---------------------------------------------
| Redis | 使用内存存储（in-memory）的非关系数据库 | 字符串，列表，集合，散列表，有序集合 | 每种数据类型都有自己的专属命令，另外还有批量操作和不完全的事务支持 | 发布与订阅、主从复制、持久化、脚本（存储过程）
| memcached | 使用内存存储的键值缓存 | 键值之间的映射 | 创建命令，读取命令，更新命令，删除命令以及其他几个命令 | 为提升性能而设的多线程服务器
| MySQL | 关系数据库 | 每个数据库可以包含多个表，每个表可以包含多个行：可以处理多个表的视图：支持空间和第三方扩展 | SELECT、INSERT、UPDATE、DELETE、函数、存储过程 | 支持 ACID 性质（需要使用 InnoDB），主从复制和主主复制
| PostgreSQL | 关系数据库 | 每个数据库可以包含多个表，每个表可以包含多个行：可以处理多个表的视图：支持空间和第三方扩展：支持可定制类型 | SELECT、INSERT、UPDATE、DELETE、内置函数、自定义的存储过程 | 支持 ACID 性质，主从复制，由第三方支持的多主复制
| MongoDB | 使用硬盘存储（on-disk）的非关系文档存储 | 每个数据库可以包含多个表，每个表可以包含多个无 schema 的 BSON 文档 | 创建命令、读取命令、更新命令、删除命令、条件查询命令等 | 支持 map-reduce 操作，主从复制，分片，空间搜索

## Redis 提供的 5 种结构

| 结构类型 | 结构存储的值 | 结构的读写能力 |
|:--------------------------------------
| STRING | 可以是字符串、整数或者浮点数 | 对整个字符串或者字符串的其中一部分执行操作：对整数和浮点数执行自增（increment）或者自减（decrement）操作
| LIST | 一个链表，链表上的每个节点都包含了一个字符串 | 从链表的两端推入或者弹出元素：根据偏移量对链表进行修剪（trim）：读取单个或者多个元素：根据值查找或者移出元素
| SET | 包含字符串的无序收集器（unordered collection），并且被包含的每个字符串都是独一无二、各不相同的 | 添加、获取、移除单个元素：检查一个元素是否存在于集合中：计算交集、并集、差集：从集合中随机获取元素
| HASH | 包含键值对的无序散列表 | 添加、获取、移除单个键值对：获取所有键值对
| ZSET（有序集合） | 字符串成员（member）与浮点数分值（score）之间的有序映射，元素的排列顺序由分值的大小决定 | 添加、获取、移除单个元素：根据分值范围（range）或者成员来获取元素

# 数据结构

## 字符串 string

| 命令 | 行为
|:----------
| GET | 获取存储在给定键中的值
| SET | 设置存储在给定键中的值
| DEL | 删除存储在给定键中的值（可用于所有类型）

举个栗子

```
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> get hello
"world"
127.0.0.1:6379> del hello
(integer) 1
127.0.0.1:6379> get hello
(nil)
127.0.0.1:6379> 
```

## 列表 list

| 命令 | 行为
|:----------
| RPUSH | 将给定值推入列表的右端
| LRANGE | 获取列表在给定范围上的所有值
| LINDEX | 获取列表在给定位置上的单个元素
| LPOP | 从列表的左端弹出一个值，并返回被弹出的值


``` 
127.0.0.1:6379> rpush names jack
(integer) 1
127.0.0.1:6379> rpush names wang
(integer) 2
127.0.0.1:6379> rpush names jack
(integer) 3
127.0.0.1:6379> lrange names 0 -1 # 表示获取所有的
1) "jack"
2) "wang"
3) "jack"
127.0.0.1:6379> lindex names 1
"wang"
127.0.0.1:6379> lpop names
"jack"
127.0.0.1:6379> lrange names 0 -1
1) "wang"
2) "jack"
```

!> 列表可以存储相同的值

## 集合 set

| 命令 | 行为
|:----------
| SADD | 将给定元素添加到集合
| SMEMBERS | 返回集合包含的所有元素
| SISMEMBER | 检查给定元素是否存在于集合中
| SREM | 如果给定的元素存在于集合中，那么移除这个元素


```
127.0.0.1:6379> sadd set-key item
(integer) 1
127.0.0.1:6379> sadd set-key item2
(integer) 1
127.0.0.1:6379> sadd set-key item3
(integer) 1
127.0.0.1:6379> sadd set-key item
(integer) 0
127.0.0.1:6379> smembers set-key
1) "item3"
2) "item2"
3) "item"
127.0.0.1:6379> sismember set-key item4
(integer) 0
127.0.0.1:6379> sismember set-key item
(integer) 1
127.0.0.1:6379> srem set-key item2
(integer) 1
127.0.0.1:6379> srem set-key item4
(integer) 0
127.0.0.1:6379> smembers set-key
1) "item3"
2) "item"
```

## 散列 hash

| 命令 | 行为
|:----------
| HSET | 在散列中关联给定的键值对
| HGET | 获取指定散列键的值
| HGETALL | 获取散列中所有的键值对
| HDEL | 如果给定键存在于散列中，那么移除这个键

```
127.0.0.1:6379> hset hash-key sub-key1 value1
(integer) 1
127.0.0.1:6379> hset hash-key sub-key2 value2
(integer) 1
127.0.0.1:6379> hset hash-key sub-key1 value1
(integer) 0
127.0.0.1:6379> hgetall hash-key
1) "sub-key1"
2) "value1"
3) "sub-key2"
4) "value2"
127.0.0.1:6379> hdel hash-key sub-key2
(integer) 1
127.0.0.1:6379> hdel hash-key sub-key2
(integer) 0
127.0.0.1:6379> hget hash-key sub-key1
"value1"
127.0.0.1:6379> hgetall hash-key
1) "sub-key1"
2) "value1"
127.0.0.1:6379> 
```

## 有序集合 zset

有序集合也是存储键值对的，有序集合中的键被称为成员（member），每个成员都是各不相同的。有序集合的值被称为分值（score），分值必须为浮点数。有序集合是 Redis 里面唯一一个既可以根据成员访问元素，又可以根据分值以及分值的排序顺序来访问元素的结构

| 命令 | 行为
|:----------
| ZADD | 将一个带有给定分值的成员添加到有序集合里面 
| ZRANGE | 根据元素在有序排列中所处的位置．从有序集合里面获取多个元素 
| ZRANGEBYSCORE | 获取有序集合在给定分值范围内的所有元素
| ZREM | 如果给定成员存在于有序集合，那么移除这个成员 

```
127.0.0.1:6379> zadd zset-key 728 member1
(integer) 1
127.0.0.1:6379> zadd zset-key 982 member0
(integer) 1
127.0.0.1:6379> zadd zset-key 982 member0
(integer) 0
127.0.0.1:6379> zrange zset-key 0 -1 withscores
1) "member1"
2) "728"
3) "member0"
4) "982"
127.0.0.1:6379> zrangebyscore zset-key 0 800 withscores
1) "member1"
2) "728"
127.0.0.1:6379> zrem zset-key member1
(integer) 1
127.0.0.1:6379> zrem zset-key member1
(integer) 0
127.0.0.1:6379> zrange zset-key 0 -1 withscores
1) "member0"
2) "982"
```