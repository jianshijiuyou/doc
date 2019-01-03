# 通用命令

| 命令 | 行为
|:------|:----
| `keys <pattern>` | 根据模式返回指定的 key。 <br> `keys *` <br> `keys ha*` <br> 时间复杂度 O(n)
| `dbsize` | 返回 key 的总数
| `exists <key>` | 判断 key 是否存在
| `del <key> [key]...` | 删除指定 key
| `expire <key> <seconds>` | 设置 key 的在多少秒后过期
| `pexpire <key> <millisecaonds>` | 设置 key 的在多少毫秒后过期
| `expireat <key> <timestamp>` | 将过期时间设置为给定的 UNIX 时间戳
| `pexpireat <key> <timestamp-milliseconds>` | 将过期时间设置为给定的 UNIX 时间戳（毫秒级）
| `ttl <key>` | 查看 key 剩余的过期时间（秒）<br> `ttl <key>`: <br>返回 `-2` 表示 key 已经过期，被删除了。<br>返回 `-1` 表示数据依然存在
| `pttl <key>` | 查看 key 剩余的过期时间（毫秒）
| `persist <key>` | 去掉 key 的过期时间
| `type <key>` |  检查 key 的数据类型

# Redis 提供的 5 种数据结构

| 结构类型 | 结构存储的值 | 结构的读写能力 |
|:-------|:--------------|:-----------------
| STRING | 可以是字符串、整数或者浮点数 | 对整个字符串或者字符串的其中一部分执行操作：对整数和浮点数执行自增（increment）或者自减（decrement）操作
| LIST | 一个链表，链表上的每个节点都包含了一个字符串 | 从链表的两端推入或者弹出元素：根据偏移量对链表进行修剪（trim）：读取单个或者多个元素：根据值查找或者移出元素
| SET | 包含字符串的无序收集器（unordered collection），并且被包含的每个字符串都是独一无二、各不相同的 | 添加、获取、移除单个元素：检查一个元素是否存在于集合中：计算交集、并集、差集：从集合中随机获取元素
| HASH | 包含键值对的无序散列表 | 添加、获取、移除单个键值对：获取所有键值对
| ZSET（有序集合） | 字符串成员（member）与浮点数分值（score）之间的有序映射，元素的排列顺序由分值的大小决定 | 添加、获取、移除单个元素：根据分值范围（range）或者成员来获取元素

## 字符串 string

再 Redis 中，字符串可以存储：字符串、整数、浮点数。

| 命令 | 行为
|:-----|:-----
| `get <key>` | 获取存储在给定键中的值
| `set <key> <val>` | 设置存储在给定键中的值
| `setnx <key> <val>` | key 不存在才设置值
| `set <key> <val> xx` | key 存在才设置值
| `del <key>` | 删除存储在给定键中的值（可用于所有类型）
| `incr <key>` | key 自增 1，如果 key 不存在 ，则自增后 `get <key>` 等于 1
| `decr <key>` | key 自减 1，如果 key 不存在 ，则自减后 `get <key>` 等于 -1
| `incrby <key> <k>` | key 自增 k，如果 key 不存在 ，则自增后 `get <key>` 等于 k
| `decrby <key> <k>` | key 自减 k，如果 key 不存在 ，则自减后 `get <key>` 等于 -k
| `incrbyfloat <key> <k>` | 增加 key 对应的浮点值 k
| `mset <key1> <val1> <key2> <val2> ...` | 批量设置
| `mget <key1> <key2>` | 批量获取
| `getset <key> <newval>` | 设置新的值同时返回旧的值
| `append <key> <val>` | 将新的 val 追加到旧 val 后，就是字符串追加
| `strlen <key>` | 计算字符串的长度（一个汉字会返回 3）
| `getrange <key> <start> <end>` |  获取指定下标的子字符串
| `setrange <key> <index> <val>` | 设置指定下标的子字符串，会覆盖原有的


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
|:-----|:-----
| `rpush <key> <val1> <val2> ...` | 将给定值推入列表的右端
| `linsert <key> before/after <val> <newval>` | 在 list 指定的值前面（后面）插入 newval
| `lrange <key> <start> <end>` | 获取列表在给定范围上的所有值，包括 end
| `lindex <key> <index>` | 获取列表在给定位置上的单个元素
| `lpop <key>` | 从列表的左端弹出一个值，并返回被弹出的值
| `lrem <key> <count> <val> ` | 根据 count 的值，从列表中删除所有和 value 相等的项 <br><br> (1) count>0 ，从左到右，<br>删除最多 count 个和 value 相等的项 <br><br> (2) count<0 ，从右到左，<br>删除最多 Math.abs(count) 个和 value 相等的项 <br><br> (3) count=0 ，删除所有和 value 相等的项
| `ltrim <key> <start> <end>` | 按照索引范围修剪列表，删除索引范围外的元素
| `llen <key>` | 计算列表的长度
| `lset <key> <index> <newval>` | 设置列表指定索引值为 newval
| `blpop <key> <timeout>` | lpop 阻塞版，timeout 为阻塞时间
| `brpop <key> <timeout>` | rpop 阻塞版
| `rpoplpush <source-key> <dest-key>` | 从 source-key 列表中弹出最右边的值并插入 dest-key 列表的左边，并向用户返回这个值
| `brpoplpush <source-key> <dest-key> <timeout>` | rpoplpush 阻塞版

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
|:-----|:-----
| `sadd <key> <val>` | 将给定元素添加到集合
| `smembers <key>` | 返回集合包含的所有元素
| `scard <key>` | 计算集合中的元素总数
| `srandmember <key> <count>` | 随机返回 count 个元素
| `spop <key> ` | 随机弹出一个元素
| `sismember <key> <val>` | 检查给定元素是否存在于集合中
| `srem <key> <val>` | 如果给定的元素存在于集合中，那么移除这个元素
| `smove <source-key> <dest-key> <val>` | 移动集合中的元素
| `sdiff <key1> <key2>` | 差集
| `sdiffstore <dest-key> <key1> <key2> ...` | 将 key1、key2 的差集存储到 dest-key 中
| `sinter <key1> <key2>` | 交集
| `sinterstore <dest-key> <key1> <key2> ...` | 将 key1、key2 的交集存储到 dest-key 中
| `sunion <key1> <key2>` | 并集
| `sunionstore <dest-key> <key1> <key2> ...` | 将 key1、key2 的并集存储到 dest-key 中

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

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/images/4b99e03b-d969-4aa6-a8cf-5bb7454dc185.png)
> 图片来自慕课网

| 命令 | 行为
|:----|:------
| `hset <key> <field> <val>` | 在散列中关联给定的键值对
| `hget <key> <field>` | 获取指定散列键的值
| `hgetall <key>` | 获取散列中所有的键值对
| `hvals <key>` | 返回 hash key 对应的所有 field 的 value
| `hkeys <key>` | 返回 hash key 对应的所有 field
| `hdel <key> <field>` | 如果给定键存在于散列中，那么移除这个键
| `hexists <key> <field>` | 判断 hash key 是否有 field
| `hlen <key>` | 计算 hash key field 的数量
|  `hmget <key> <field1> <field2> ...` | 批量获取
|  `hmset <key> <field1> <val1> <field2> <val2> ...` | 批量设置
| `hsetnx <key> <field> <val>` | 已经存在，则失败
| `hincrby <key> <field> <k>` | 对应的 field 自增 k（int）
| `hincrbyfloat <key> <field> <k>` | 对应的 field 自增 k（float）

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
|:-----|:-----
| `zadd <key> <score> <val> ...` | 将一个带有给定分值的成员添加到有序集合里面 
| `zrange <key> <start_score> <end_score> [withscores]` | 根据元素在有序排列中所处的位置．从有序集合里面获取多个元素 
| `zrangebyscore <key> <min_score> <max_score> [withscores]` | 获取有序集合在给定分值范围内的所有元素
| `zrem <key> <val>` | 如果给定成员存在于有序集合，那么移除这个成员 
| `zremrangebyrank <key> <start> <end>` | 删除指定排名内的升序元素
| `zremrangebyscore <key> <start> <end>` | 删除指定 score 内的升序元素
| `zscore <key> <val>` | 返回元素的 score
| `zincrby <key> <increScore> <val>` | 将元素 val 的 score 加上 increScore
| `zcard <key>` | 返回元素的总数
| `zrank <key> <val>` | 获取 val 的排名，升序
| `zcount <key> <min_score> <max_score>` | 返回有序集合内在指定 score 范围内的个数
| `zrevrank <key> <member>` | 获取 member 的排名，降序
| `zrevrange <key> <start> <stop> [withscores]` | 返回给定排名范围的成员，降序
| `zrevrangebyscore <key> <min> <max> [withscores]` | zrangebyscore 降序版
| `zinterstore` | 交集
| `zunionstore` | 并集

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

# 发布与订阅

| 命令 | 行为
|:----|:------
| `subscribe <channel> [channel ...]` | 订阅给定的一个或多个频道
| `unsubscribe [channel] [channel ...]` | 退订给定的一个或多个频道，如果没有给出频道，则退订所有频道
| `publish <channel> <message>` | 向给定频道发送消息
| `psubscribe <pattern> [pattern ...]` | 订阅与给定模式相匹配的所有频道
| `punsubscribe [pattern] [pattern ...]` | 退订给定的模式，如果没有给出任何模式，则退订所有模式

# 事务

看一个问题

``` python
import redis
import time
import threading


client = redis.Redis()

def notrans():
    result = client.incr('notrans:')
    print(result)
    time.sleep(.1)
    client.incr('notrans:', -1)

for i in range(3):
    threading.Thread(target=notrans).start()
time.sleep(.5)

# 1
# 2
# 3
```

因为没有使用事务，三个线程都在自减操作之前执行了，不符合预期。

下面看一个使用了事务的版本：

``` python
import redis
import time
import threading


client = redis.Redis()

def notrans():
    pipeline =client.pipeline()
    pipeline.incr('trans:')
    time.sleep(.1)
    pipeline.incr('trans:', -1)
    result = pipeline.execute()
    print(result)

for i in range(3):
    threading.Thread(target=notrans).start()
time.sleep(.5)

# [1, 0]
# [1, 0]
# [1, 0]
```

ok，结果符合预期

如果在调用 `client.pipeline()` 方法时传入 `False` 就可以不启用事务，并且还能将一系列命令打包一次性发送给 Redis 服务端。从而提高性能。

# 持久化

「快照持久化」和「AOF 持久化」可以同时使用

## 快照持久化

| 配置选项 | 说明
|:--------|:-----
| save | 例：`save 60 1000` <br><br> 从 Redis 最近一次创建快照之后开始算起，当「`60` 秒之内有 `1000` 次写入」这个条件被满足时，Redis 就会自动触发 `bgsave` 命令。<br><br> `save` 选项可配置多组，例：`save 900 1 60 1000`
| stop-writes-on-bgsave-error | 例：`stop-writes-on-bgsave-error on`
| rdbcompression | 例：`rdbcompression yes`
| dbfilename | 例：`dbfilename dump.rdb` <br><br> 指定存储快照的文件名，这个文件会存储在 `dir` 指定的路径下。

创建快照的办法有以下几种：

1. 发送 `bgsave` 命令：

    *通过子进程将快照写入磁盘，不影响父进程处理命令请求。*

2. 发送 `save` 命令：

    *主进程直接创建快照，期间不处理其他命令，不常用。*

3. 设置 `save` 配置选项：

    *满足条件则触发 `bgsave` 命令。*

4. 当 Redis 通过 `shutdown` 命令接收到关闭服务器的请求时或者接收到标准 `TERM` 信号时：

    *执行 `save` 命令，并在执行完毕后关闭服务器。*

5. 当一个 Redis 服务器连接另一个 Redis 服务器，并向对方发送 `sync` 命令来开始一次复制操作的时候：

    *如果主服务器目前没有在执行 `bgsave` 操作，或者主服务器并非刚刚执行完 `bgsave` 操作，那么主服务器就会执行 `bgsave` 命令。*


!> 在仅使用「快照持久化」时，如果系统发生崩溃，Redis 将丢失最近一次生成快照之后更改的所有数据。



## AOF(append-only file) 持久化

| 配置选项 | 说明
|:---------|:----
| appendonly | 例：`appendonly yes` <br><br> 开启或关闭「AOF 持久化」
| appendfsync | 例：`appendfsync` [ `always` &#124; `everysec` &#124; `no` ] <br><br> `always`: 每个 Redis 写命令都要同步写入硬盘。这种做法会严重降低 Redis 的速度<br><br> `everysec`: 每秒执行一次同步，显示地将多个写命令同步到硬盘（推荐）<br><br> `no`: 让操作系统决定何时进行同步
| no-appendfsync-on-rewrite | 例：`no-appendfsync-on-rewrite no`
| auto-aof-rewrite-percentage | 例：`auto-aof-rewrite-percentage 100` <br><br> AOF 文件比上一次重写之后的体积大了 `100%`（一倍）并且满足 `auto-aof-rewrite-min-size` 选项时则进行 AOF 文件重写。
| auto-aof-rewrite-min-size | 例：`auto-aof-rewrite-min-size 64mb` <br><br> 当 AOF 文件大于 `64mb` 并且满足 `auto-aof-rewrite-percentage` 选项时则进行 AOF 文件重写.

AOF 文件会越来越大，使用 `bgrewriteaof` 命令可以重写 AOF 文件，移除其中的冗余命令。`bgrewriteaof` 命令也是通过创建子进程来完成的。

如果 AOF 文件的达到数十 GB，那重写时删除这么大的文件可能会导致操作系统挂起（hang）数秒。

共享选项

| 选项 | 说明
|:-----|:--------
| dir | 例：`dir ./` <br><br> 这个选项决定了「快照文件」和 「AOF 文件」的保存位置

# 复制

主从复制之前，请确保配置了 `dir` 选项和 `dbfilename` 选项，并且两个选项指定的路径和文件对于 Redis 进程来说都是可写的。

开启从服务器的必须配置选项只有一个：

`slaveof <host> <port>`

如果 Redis 服务器在启动时指定了一个包含上述配置的配置文件，那么 Redis 服务器将根据该选项给定的 IP 地址和端口号来连接主服务器。对于一个正在运行的 Redis 服务器，用户可以通过发送 `slaveof no one` 命令来让服务器终止复制操作：也可以通过发送 `slaveof host port` 命令来让服务器开始复制一个新的主服务器。

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/images/20180611163918572.png)

!> 从服务器在进行同步时，会清空自己的所有数据

!> Redis 不支持主主复制

!> 实际中最好让主服务器只使用 50%~65% 的内存，留下剩余内存用于执行 `bgsave` 命令和创建记录写命令的缓冲区

当多个从服务器尝试连接同一个主服务器的时候，会出现两种情况。

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/images/20180611164904277.png)

还可以配置如下的 Redis 主从链：

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/images/20180611165211503.png)

# redis-py 使用

redis-py 库提供了两个类 Redis 和 StrictRedis 来实现 Redis 的命令操作。

推荐使用 StrictRedis，它和 Redis 命令参数表现一致。

``` python
from redis import StrictRedis

redis = StrictRedis()
redis.set('name', 'Bob')
print(redis.get('name'))
```

推荐阅读 [https://cuiqingcai.com/5587.html](https://cuiqingcai.com/5587.html)

# redis-cli

单独下载 redis-cli

``` bash
sudo apt-get install redis-tools
```

使用

``` bash
Usage: redis-cli [OPTIONS] [cmd [arg [arg ...]]]
  -h <hostname>      Server hostname (default: 127.0.0.1).
  -p <port>          Server port (default: 6379).
  -a <password>      Password to use when connecting to the server.
  -u <uri>           Server URI.
```
## 举个栗子

``` bash
$ redis-cli -u redis://:password@127.0.0.1:6379/0

# or
$ redis-cli -h 127.0.0.1 -p 6379 -a password

#or
# -n <db number>
$ redis-cli -h 127.0.0.1 -p 6379 -n 1 -a password
# or
$ redis-cli -h 127.0.0.1 -p 6379           
111.230.231.89:6379> keys *
(error) NOAUTH Authentication required.
111.230.231.89:6379> auth password
OK
111.230.231.89:6379> keys *
1) "aaa"
2) "a"
```