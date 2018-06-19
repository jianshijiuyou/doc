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

## 单线程模型

Redis 是单线程模型，就是同一时间只能执行一个命令

# 安装

安装编译工具 gcc

```
sudo apt update
sudo apt install make gcc
```

下载 & 编译

```
wget http://download.redis.io/releases/redis-[stable].tar.gz
tar xvzf redis-[stable].tar.gz
cd redis-stable
make
```

此时，您可以尝试通过键入 `make test` 来正确工作，但这是可选步骤。编译之后，Redis 发行版内部的 src 目录将填充 Redis 中不同的可执行文件：

* `redis-server` 是 Redis 服务器本身。
* `redis-sentinel` 是 Redis Sentinel 可执行文件（监视和故障转移）。
* `redis-cli` 是与 Redis 交互的命令行程序。
* `redis-benchmark` 用于检查 Redis 的性能.
* `redis-check-aof` 和 `redis-check-dump` 在罕见的数据文件损坏事件中很有用。

在适当的位置复制 Redis server 和 command line interface 是一个好主意，可以手动使用以下命令：

* `sudo cp src/redis-server /usr/local/bin/`
* `sudo cp src/redis-cli /usr/local/bin/`

或者只是使用 `sudo make install`。

## 启动

启动 Redis 服务器的最简单方法就是不带任何参数地执行 redis-server 二进制文件。

```
redis-server
```

如果在生产环境，应该使用配置位置文件

```
redis-server /etc/redis.conf
```

## 检查 Redis 是否正在工作

```
$ redis-cli ping
PONG
```

## 更正确地安装 Redis

创建一个存储 Redis 配置文件和数据的目录：

```
sudo mkdir /etc/redis
sudo mkdir /var/redis
```

将在 utils 目录下的 init 脚本复制到 `/etc/init.d` 中。我们建议使用您运行 Redis 实例的端口名称来调用它。例如：

```
sudo cp utils/redis_init_script /etc/init.d/redis_6379
```

编辑 init 脚本

```
sudo vim /etc/init.d/redis_6379
```

确保相应地将 `REDISPORT` 修改为您正在使用的端口。 pid 文件路径和配置文件名都取决于端口号。

* 在 Redis 源码包的根目录中找到的模板配置文件复制到 `/etc/redis/` 中，使用端口号作为名称，例如：

    ```
    sudo cp redis.conf /etc/redis/6379.conf
    ```

* 在 `/var/redis` 中创建一个可用作此 Redis 实例的数据和工作目录的目录：

    ```
    sudo mkdir /var/redis/6379
    ```

* 编辑配置文件，确保执行以下更改：

    * 将 `daemonize` 设置为 `yes`（默认情况下，它设置为 `no`）。
    * 将 `pidfile` 设置为 `/var/run/redis_6379.pid`（如果需要，修改端口）。
    * 相应地更改 `port`。在我们的示例中，它不是必需的，因为默认端口已经是 `6379`。
    * 设置您的首选 `loglevel` 级别。
    * 将 `logfile` 设置为 `/var/log/redis_6379.log`
    * 将 `dir` 设置为 `/var/redis/6379`（非常重要的一步！）

 * 最后，使用以下命令将新的 Redis init 脚本添加到所有默认运行级别：

 ```
 sudo update-rc.d redis_6379 defaults
 ```

 !> 如果出现警告 `insserv: warning: script 'redis_6379' missing LSB tags and overrides`，别担心，并不影响。

你完成了！现在您可以尝试运行您的实例：

```
sudo /etc/init.d/redis_6379 start
```

确保一切按预期工作：

* 尝试使用redis-cli 来 ping 您的实例。
* 使用 `redis-cli save` 进行测试保存并检查转储文件是否已正确存储到 `/var/redis/6379/` 中（您应该找到一个名为 `dump.rdb` 的文件）。
* 检查您的 Redis 实例是否正确登录到日志文件中。
* 如果它是一台可以毫无问题地尝试使用的新机器，请确保在重启后一切仍然正常。

## 常用配置

| key | 说明 |
|:------------
| daemonize | 是否是守护进程 （`no` 或 `yes`）
| port | Redis 对外端口号
| logfile | Redis 系统日志文件
| dir | Redis 工作目录

> 通过在 redis-cli 中执行 `config get *` 查看所有的配置信息

## 在 Python 中使用 Redis

安装 Python 操作 Redis 的库

```
sudo pip install redis hiredis
```
!> hiredis 包是一个 C 库，它可以提高 Python 的 Redis 客户端库的速度。

测试

``` python
In [1]: import redis

In [2]: client = redis.StrictRedis(host='127.0.0.1', port='6379')

In [3]: set_result = client.set('hello', 'python-redis')

In [4]: set_result
Out[4]: True

In [5]: value = client.get('hello')

In [6]: value
Out[6]: b'python-redis'

```