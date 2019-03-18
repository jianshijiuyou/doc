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
