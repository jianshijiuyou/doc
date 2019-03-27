# 基本 API

`create/path data`

创建一个名为 `/path` 的 `znode` 节点，并包含数据 `data`。

`delete/path`

删除名为 `/path` 的 `znode`。

`exists/path`

检查是否存在名为 `/path` 的节点。

`setData/path data`

设置名为 `/path` 的 `znode` 的数据为 `data`。

`getData/path`

返回名为 `/path` 节点的数据信息。

`getChildren/path`

返回所有 `/path` 节点的所有子节点列表。

!> 需要注意的是，ZooKeeper 并不允许局部写入或读取 `znode` 节点的数据。当设置一个 `znode` 节点的数据或读取时， `znode` 节点的内容会被整个替换或全部读取进来。

# 节点类型

一共有四种节点

**持久节点和临时节点**

`znode` 节点可以是持久（persistent）节点，还可以是临时（ephemeral）节点。持久的 `znode`，如 `/path` ，只能通过调用 `delete` 来进行删除。临时的 `znode` 与之相反，当创建该节点的客户端崩溃或关闭了与 ZooKeeper 的连接时，这个节点就会被删除。

一个临时 `znode` ，在以下两种情况下将会被删除：

* 当创建该 `znode` 的客户端的会话因超时或主动关闭而中止时。
* 当某个客户端（不一定是创建者）主动删除该节点时。

**持久有序的（persistent_sequential）和临时有序的（ephemeral_sequential）节点**

一个 `znode` 还可以设置为有序（sequential）节点。一个有序 `znode` 节点被分配唯一个单调递增的整数。当创建有序节点时，一个序号会被追加到路径之后。例如，如果一个客户端创建了一个有序 `znode` 节点，其路径为 `/tasks/task-` ，那么 ZooKeeper 将会分配一个序号，如 `1`，并将这个数字追加到路径之后，最后该 `znode` 节点为 `/tasks/task-1` 。有序 `znode` 通过提供了创建具有唯一名称的 `znode` 的简单方式。同时也通过这种方式可以直观地查看 `znode` 的创建顺序。

# 安装

http://zookeeper.apache.org

``` bash
tar -zxvf zookeeper-3.4.12.tar.gz
```

在解压后的项目更目录下执行以下操作

修改配置文件名字

``` bash
mv conf/zoo_sample.cfg conf/zoo.cfg
```

如果需要修改数据存储路径，可以在配置文件中修改

```
dataDir=/users/me/zookeeper
```

启动服务（后台运行）

``` bash
bin/zkServer.sh start
```

停止服务

``` bash
bin/zkServer.sh stop
```

启动服务（前台运行）

``` bash
bin/zkServer.sh start-foreground
```

启动客户端

``` bash
bin/zkCli.sh
```

常用命令

``` bash
[zk: localhost:2181(CONNECTED) 3] ls /
[zookeeper]
[zk: localhost:2181(CONNECTED) 4] create /workers "" # create -e /master 临时节点
Created /workers
[zk: localhost:2181(CONNECTED) 5] ls /
[zookeeper, workers]
# get /workers 获取节点的元数据和数据
[zk: localhost:2181(CONNECTED) 6] delete /workers
[zk: localhost:2181(CONNECTED) 7] ls /
[zookeeper]
[zk: localhost:2181(CONNECTED) 8] quit
Quitting...
2018-09-15 16:24:32,728 [myid:] - INFO  [main:ZooKeeper@687] - Session: 0x100001b72600000 closed
2018-09-15 16:24:32,728 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread@521] - EventThread shut down for session: 0x100001b72600000
```

# 状态

一共有四种状态

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/images/20180915163142.png)

# 命令

| 命令 | 说明 | 
|:-----|:--------
| `zkCli.sh -server 127.0.0.1:2181` | 连接服务器
| `ls /` | 列出根目录下的所有节点
| `create /workers "data"` | 创建节点和数据
| `create -e /workers "data"` | -e 临时节点
| `set /workers "data"` | 修改节点数据 <br/> 可以不加 `""` 放个 JSON 字典，但是不能有换行符 <br /> `set /workers {"name":"jack","pwd":"123"}`
| `get /workers` | 获取节点的元数据和数据
| `delete /workers` | 删除节点
| `stat /master true` | 查看节点属性，`true` 代表监视节点变化
| `quit` | 退出客户端

# 四字命令

> [官网](https://zookeeper.apache.org/doc/r3.4.8/zookeeperAdmin.html)

ZooKeeper 支持某些特定的四字命令字母与其的交互。它们大多是查询命令，用来获取 ZooKeeper 服务的当前状态及相关信息。用户在客户端可以通过 `telnet` 或 `nc` 向 ZooKeeper 提交相应的命令

三个更有趣的命令：“stat” 给出了关于服务器和连接客户端的一些一般信息，而 “srvr” 和 “cons” 分别给出了服务器和连接的扩展细节。

用法示例

``` bash
echo stat | nc 127.0.0.1 2181
```

| 命令 | 说明 | 
|:--|:--|
| conf | **New in 3.3.0:** 打印有关服务配置的详细信息。
| cons | **New in 3.3.0:** 列出连接到此服务器的所有客户端的完整连接/会话详细信息。包括有关接收/发送的数据包数量，会话 ID，操作延迟，上次执行的操作等信息...
| crst | **New in 3.3.0:** 重置所有连接的连接/会话统计信息。
| dump | 列出未完成的会话和临时节点。这只适用于 leader。
| envi | 打印服务环境的详细信息
| ruok | 测试服务器是否在非错误状态下运行。如果正在运行，服务器将使用 imok 响应。否则它根本不会响应。<br><br> “imok” 的响应不一定表示服务器已加入仲裁，只是服务器进程处于活动状态并绑定到指定的客户端端口。有关状态 wrt 仲裁和客户端连接信息的详细信息，请使用 “stat” 。
| srst | 重置服务器统计。
| srvr | **New in 3.3.0:** 列出服务器的完整详细信息。
| stat | 列出服务器和连接客户端的简要详细信息。
| wchs | **New in 3.3.0:** 列出服务器 watch 的简要信息。
| wchc | **New in 3.3.0:** 按会话列出有关服务器 watch 的详细信息。这将输出与相关 watch（路径）的会话（连接）列表。请注意，根据 watch 的数量，此操作可能很昂贵（即影响服务器性能），请谨慎使用。
| wchp | **New in 3.3.0:** 按路径列出有关服务器 watch 的详细信息。这将输出一个包含相关会话的路径列表（znodes）。请注意，根据 watch 的数量，此操作可能很昂贵（即影响服务器性能），请谨慎使用。
| mntr | **New in 3.4.0:** 输出可用于监视群集运行状况的变量列表。

## conf

``` bash
$ echo conf | nc 127.0.0.1 2181
clientPort=2181                 # 客户端端口号 
dataDir=/data/version-2         # 数据文件目录
dataLogDir=/datalog/version-2   # 日志文件目录  
tickTime=2000                   # 间隔单位时间
maxClientCnxns=60               # 最大连接数  
minSessionTimeout=4000          # 最小session超时
maxSessionTimeout=40000         # 最大session超时  
serverId=1                      # id  
initLimit=5                     # 初始化时间  
syncLimit=2                     # 心跳时间间隔  
electionAlg=3                   # 选举算法 默认 3  
electionPort=3888               # 选举端口  
quorumPort=2888                 # 法人端口  
peerType=0                      # 未确认
```

## cons

``` bash
$ echo cons | nc 127.0.0.1 2181
 /127.0.0.1:50586[1](queued=0,recved=19107,sent=19108,sid=0x1001784d1f5006a,lop=PING,est=1553605379960,to=10000,lcxid=0x34,lzxid=0xffffffffffffffff,lresp=481778503,llat=0,minlat=0,avglat=0,maxlat=8)

# ip=ip
# port=端口
# queued=所在队列
# received=收包数
# sent=发包数
# sid=session id
# lop=最后操作
# est=连接时间戳
# to=超时时间
# lcxid=最后id(未确认具体id)
# lzxid=最后id(状态变更id)
# lresp=最后响应时间戳
# llat=最后/最新 延时
# minlat=最小延时
# maxlat=最大延时
# avglat=平均延时
```

## envi

``` bash
$ echo envi | nc 127.0.0.1 2181
Environment:
zookeeper.version=...                           # 版本
host.name=587c84529768                          # host 信息
java.version=1.8.0_191                          # java 版本
java.vendor=Oracle Corporation                  # 供应商
java.home=/usr/lib/jvm/java-1.8-openjdk/jre     # jdk目录
java.class.path=...                             # classpath
java.library.path=...                           # lib path
java.io.tmpdir=/tmp                             # temp目录
java.compiler=<NA>
os.name=Linux
os.arch=amd64
os.version=3.10.0-957.5.1.el7.x86_64
user.name=zookeeper
user.home=/home/zookeeper
user.dir=/zookeeper-3.4.13
```

## srvr

``` bash
$ echo srvr | nc 127.0.0.1 2181
Zookeeper version: ...          # 版本
Latency min/avg/max: 0/0/38     # 延时
Received: 307693                # 收包
Sent: 308300                    # 发包
Connections: 21                 # 连接数
Outstanding: 0                  # 堆积数
Zxid: 0x500004664               # 操作id
Mode: follower                  # leader/follower
Node count: 1706                # 节点数
```

## mntr

``` bash
$ echo mntr | nc 127.0.0.1 2181
zk_version	...                             # 版本
zk_avg_latency	0                           # 平均延时
zk_max_latency	38                          # 最大延时
zk_min_latency	0                           # 最小延时
zk_packets_received	308605                 # 收包数
zk_packets_sent	309212                     # 发包数
zk_num_alive_connections	21                # 连接数
zk_outstanding_requests	0                  # 堆积请求数
zk_server_state	follower                   # 状态 leader/follower
zk_znode_count	1706                        # znode 数量
zk_watch_count	97                          # watch 数量
zk_ephemerals_count	123                    # 临时节点（znode）
zk_approximate_data_size	1818327           # 数据大小
zk_open_file_descriptor_count	51           # 打开的文件描述符数量
zk_max_file_descriptor_count	1048576       # 最大文件描述符数量
zk_fsync_threshold_exceed_count	0
```

# 可视化工具

https://github.com/DeemOpen/zkui