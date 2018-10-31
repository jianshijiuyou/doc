> [http://www.mongodb.org.cn](http://www.mongodb.org.cn)

# 关系型数据库遵循 ACID 规则

事务在英文中是 transaction，和现实世界中的交易很类似，它有如下四个特性：

| 特性 | 含义 |
|:--------------
| A (Atomicity) 原子性 | 原子性很容易理解，也就是说事务里的所有操作要么全部做完，要么都不做，事务成功的条件是事务里的所有操作都成功，只要有一个操作失败，整个事务就失败，需要回滚。 <br><br> 比如银行转账，从A账户转100元至B账户，分为两个步骤：1）从A账户取100元；2）存入100元至B账户。这两步要么一起完成，要么一起不完成，如果只完成第一步，第二步失败，钱会莫名其妙少了100元。
| C (Consistency) 一致性 | 一致性也比较容易理解，也就是说数据库要一直处于一致的状态，事务的运行不会改变数据库原本的一致性约束。<br><br> 例如现有完整性约束a+b=10，如果一个事务改变了a，那么必须得改变b，使得事务结束后依然满足a+b=10，否则事务失败。
| I (Isolation) 独立性 | 所谓的独立性是指并发的事务之间不会互相影响，如果一个事务要访问的数据正在被另外一个事务修改，只要另外一个事务未提交，它所访问的数据就不受未提交事务的影响。<br><br> 比如现有有个交易是从A账户转100元至B账户，在这个交易还未完成的情况下，如果此时B查询自己的账户，是看不到新增加的100元的。
| D (Durability) 持久性 | 持久性是指一旦事务提交后，它所做的修改将会永久的保存在数据库上，即使出现宕机也不会丢失。

# 分布式计算的优缺点

| 优点 | - |
|:-------------
| 可靠性（容错） | 分布式计算系统中的一个重要的优点是可靠性。一台服务器的系统崩溃并不影响到其余的服务器。
| 可扩展性 | 在分布式计算系统可以根据需要增加更多的机器。
| 资源共享 | 共享数据是必不可少的应用，如银行，预订系统。
| 灵活性 | 由于该系统是非常灵活的，它很容易安装，实施和调试新的服务。
| 更快的速度 | 分布式计算系统可以有多台计算机的计算能力，使得它比其他系统有更快的处理速度。
| 开放系统 | 由于它是开放的系统，本地或者远程都可以访问到该服务。
| 更高的性能 | 相较于集中式计算机网络集群可以提供更高的性能（及更好的性价比）。
| **缺点** | -
| 故障排除 | 故障排除和诊断问题。
| 软件 | 更少的软件支持是分布式计算系统的主要缺点。
| 网络 | 网络基础设施的问题，包括：传输问题，高负载，信息丢失等。
| 安全性 | 开发系统的特性让分布式计算系统存在着数据的安全性和共享的风险等问题。

# CAP 定理（CAP theorem）

在计算机科学中, CAP定理（CAP theorem）, 又被称作 布鲁尔定理（Brewer's theorem）, 它指出对于一个分布式计算系统来说，不可能同时满足以下三点:

* **一致性(Consistency)** (所有节点在同一时间具有相同的数据)
* **可用性(Availability)** (保证每个请求不管成功或者失败都有响应)
* **分隔容忍(Partition tolerance)** (系统中任意信息的丢失或失败不会影响系统的继续运作)

CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，最多只能同时较好的满足两个。

因此，根据 CAP 原理将 NoSQL 数据库分成了满足 CA 原则、满足 CP 原则和满足 AP 原则三 大类：

* CA - 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。
* CP - 满足一致性，分区容忍必的系统，通常性能不是特别高。
* AP - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/images/71c3942d-c286-4694-8fcd-4cbd3e54f4e4.png)

# MongoDB 概念解析

在 mongodb 中基本的概念是文档、集合、数据库


|SQL术语/概念|	MongoDB术语/概念	|解释/说明
|:----------------------------------------
| database	   |database	      |数据库
| table	       |collection	      |  数据库表/集合
| row	       |     document	  |      数据记录行/文档
| column	   |     field	      |  数据字段/域
| index	       | index	          |索引
| table joins       |   	 	  |  表连接，MongoDB 不支持
| primary key |       primary key|	主键，MongoDB 自动将 `_id` 字段设置为主键

通过下图实例，我们也可以更直观的的了解Mongo中的一些概念：

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/images/5bf6919b-fe32-4fd3-ad2a-de311650e4a5.png)

!> [更多内容](http://www.mongodb.org.cn/tutorial/6.html)

## 数据类型


| 数据类型	    |   描述
|:---------------------------
| String	    |字符串。存储数据常用的数据类型。在 MongoDB 中，UTF-8 编码的字符串才是合法的。
| Integer	    |整型数值。用于存储数值。根据你所采用的服务器，可分为 32 位或 64 位。
| Boolean	    |布尔值。用于存储布尔值（真/假）。
| Double	    |双精度浮点值。用于存储浮点值。
| Min/Max keys	|将一个值与 BSON（二进制的 JSON）元素的最低值和最高值相对比。
| Arrays	    |用于将数组或列表或多个值存储为一个键。
| Timestamp	    |时间戳。记录文档修改或添加的具体时间。
| Object	    |用于内嵌文档。
| Null	        |用于创建空值。
| Symbol	    |符号。该数据类型基本上等同于字符串类型，但不同的是，<br>它一般用于采用特殊符号类型的语言。
| Date	        |日期时间。用 UNIX 时间格式来存储当前日期或时间。<br>你可以指定自己的日期时间：创建 Date 对象，传入年月日信息。
| Object ID	    |对象 ID。用于创建文档的 ID。
| Binary Data	|二进制数据。用于存储二进制数据。
| Code	        |代码类型。用于在文档中存储 JavaScript 代码。
| Regular expression	|正则

# 安装

## Red Hat & CentoS

> [https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/)

在 RHEL 或 CentOS 上安装 MongoDB Community Edition

### 概览

本教程使用 `.rpm` 软件包在 RHEL 或 CentOS 6 和 7 上安装 MongoDB Community Edition。

!> 本安装指南仅支持 64 位系统。详细信息请参见[平台支持](https://docs.mongodb.com/manual/release-notes/3.0-compatibility/#compatibility-platform-support)。<br><br> MongoDB 3.4 取消了对 RHEL 5 的支持。

### 组件（包）

MongoDB 在他们自己的仓库中提供了官方支持的包。该存储库包含以下软件包：

| 包名 | 描述 |
|:--------------
| mongodb-org | 一个元数据包，它将自动安装下面列出的四个组件包。
| mongodb-org-server | 包含 mongod 守护进程和相关的配置和 init 脚本。
| mongodb-org-mongos | 包含 mongos 守护进程。
| mongodb-org-shell | 包含 mongo shell。
| mongodb-org-tools | 包含以下 MongoDB 工具：mongoimport bsondump，mongodump，mongoexport，mongofiles，mongoperf，mongorestore，mongostat 和 mongotop。

mongodb-org-server 软件包提供了一个初始化脚本，该脚本使用 `/etc/mongod.conf` 配置文件启动 `mongod`。

默认情况下，软件包提供的默认 `/etc/mongod.conf` 配置文件的 `bind_ip` 设置为 `127.0.0.1`。在初始化副本集之前根据您的环境需要修改此设置。

### 安装

1. 配置包管理系统（yum）。

    创建一个 `/etc/yum.repos.d/mongodb-org-3.6.repo` 文件，以便您可以使用 yum 直接安装 MongoDB。

    **对于 MongoDB 3.6**

    内容如下：

    ```
    [mongodb-org-3.6]
    name=MongoDB Repository
    baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.6/x86_64/
    gpgcheck=1
    enabled=1
    gpgkey=https://www.mongodb.org/static/pgp/server-3.6.asc
    ```

    **对于 3.6 之前的 MongoDB 版本**

    要从早期版本系列（如 3.4）安装软件包，可以在存储库配置中指定版本系列。例如，要将系统限制到 3.4 版本系列，请创建`/etc/yum.repos.d/mongodb-org-3.4.repo` 文件以保存 MongoDB 3.4 存储库的以下配置信息：

    ```
    [mongodb-org-3.4]
    name=MongoDB 3.4 Repository
    baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/
    gpgcheck=0
    enabled=1
    ```

    您可以在[存储库本身](https://repo.mongodb.org/yum/redhat/?_ga=2.155976952.214965866.1529389993-1180781705.1528335737)中找到每个版本的 `.repo` 文件。请记住，奇数次版本（例如3.5）是开发版本，不适合生产使用。

2. 安装 MongoDB 软件包。

    要安装最新的稳定版本的 MongoDB，请发出以下命令：

    ```
    sudo yum install -y mongodb-org
    ```

    要安装特定版本的 MongoDB，请分别指定每个组件包并将版本号附加到包名称，如下例所示：

    ```
    sudo yum install -y mongodb-org-3.6.5 mongodb-org-server-3.6.5 mongodb-org-shell-3.6.5 mongodb-org-mongos-3.6.5 mongodb-org-tools-3.6.5
    ```

    您可以指定任何可用的 MongoDB 版本。然而，yum 会在新版本可用时升级软件包。要固定软件包，请将以下 `exclude` 指令添加到 `/etc/yum.conf` 文件中：

    ```
    exclude=mongodb-org,mongodb-org-server,mongodb-org-shell,mongodb-org-mongos,mongodb-org-tools
    ```

### 运行

大多数类 Unix 操作系统会限制会话可能使用的系统资源。这些限制可能会对 MongoDB 的运营产生负面影响。有关更多信息，请参阅 [UNIX ulimit 设置](https://docs.mongodb.com/manual/reference/ulimit/)。

#### 配置 SELinux

!> 如果您使用的是 SELinux，则必须配置 SELinux 以允许 MongoDB 在基于 Red Hat Linux 的系统（RHEL 或 CentOS）上启动。

要配置 SELinux，管理员有三种选择：

* 如果 SELinux 处于 enforcing 模式，则允许访问 MongoDB 部署将使用的相关端口（例如 27017）。对于默认设置，可以通过运行一下命令来完成

    ```
    semanage port -a -t mongod_port_t -p tcp 27017
    ```

* 通过将 `/etc/selinux/config` 中的 `SELINUX` 设置设为 `disabled` 来禁用 SELinux。

    ```
    SELINUX=disabled
    ```

    您必须重新启动系统才能使更改生效。

* 通过 `/etc/selinux/config` 中的 `SELINUX` 设置设为 `permissive` 将 SELinux 设置为许可模式。

    ```
    SELINUX=permissive
    ```

    您必须重新启动系统才能使更改生效。

    您可以将 `permissive` 更改为 `setenforce` 模式。 `setenforce` 不需要重启，但不是持久的。

或者，您可以选择在安装 Linux 操作系统时不安装 SELinux 软件包，或选择删除相关软件包。

#### 数据目录和权限

!> 在 RHEL 7.0 上，如果更改数据路径，则默认的 SELinux 策略将阻止 mongod 在新数据路径上拥有写入权限，前提是您不更改安全上下文。

MongoDB 实例默认将其数据文件存储在 `/var/lib/mongo` 中，并将其日志文件存储在 `/var/log/mongodb` 中，并使用 mongod 用户帐户运行。您可以在 `/etc/mongod.conf` 中指定备用日志和数据文件目录。有关更多信息，请参阅 [systemLog.path](https://docs.mongodb.com/manual/reference/configuration-options/#systemLog.path) 和 [storage.dbPath](https://docs.mongodb.com/manual/reference/configuration-options/#storage.dbPath)。


如果更改运行 MongoDB 进程的用户，则必须修改 `/var/lib/mongo and /var/log/mongodb` 目录的访问控制权限，以使该用户可以访问这些目录。

#### 程序

1. 启动 MongoDB。

    您可以通过发出以下命令启动 mongod 进程：

    ```
    sudo service mongod start
    ```

2. 确认 MongoDB 已成功启动

    您可以通过检查 `/var/log/mongodb/mongod.log` 上日志文件的内容来验证 mongod 进程是否已成功启动

    ```
    [initandlisten] waiting for connections on port <port>
    ```

    其中 `<port>` 是 `/etc/mongod.conf` 中配置的端口，默认为 `27017`。

    您可以选择通过发出以下命令来确保 MongoDB 在启动系统后启动：

    ```
    sudo chkconfig mongod on
    ```

3. 停止 MongoDB。

    根据需要，您可以通过发出以下命令来停止 mongod 进程：

    ```
    sudo service mongod stop
    ```

4. 重启 MongoDB。

    您可以通过发出以下命令重新启动 mongod 进程：

    ```
    sudo service mongod restart
    ```

    您可以通过观察 `/var/log/mongodb/mongod.log` 文件中的输出来跟踪错误或重要消息的进程状态。

5. 开始使用 MongoDB。

    在与 mongod 相同的主机上启动一个 mongo shell。使用 `--host` 命令行选项指定 mongod 侦听的本地主机地址和端口：

    ```
    mongo --host 127.0.0.1:27017
    ```

    稍后，要停止 MongoDB，请在运行 mongod 实例的终端中按 Control + C.

### 卸载

要从系统中完全删除 MongoDB，您必须删除 MongoDB 应用程序本身，配置文件以及包含数据和日志的任何目录。以下部分将指导您完成必要的步骤。

!> 此过程将彻底删除 MongoDB 及其配置和所有数据库。这个过程是不可逆的，因此请确保在继续之前备份所有配置和数据。

1. 停止 MongoDB。

    ```
    sudo service mongod stop
    ```

2. 删除软件包

    删除以前安装的任何 MongoDB 软件包。

    ```
    sudo yum erase $(rpm -qa | grep mongodb-org)
    ```

3. 删除数据目录

    删除 MongoDB 数据库和日志文件。

    ```
    sudo rm -r /var/log/mongodb
    sudo rm -r /var/lib/mongo
    ```

# 远程连接

!> 不要再生产环境下使用

修改配置文件 `/etc/mongod.conf`

```
# network interfaces
net:
  port: 27017
  bindIp: 0.0.0.0
```

重启

```
sudo service mongod restart
```

# robomongo

[robomongo](https://robomongo.org/) 是 MongoDB 的一个 GUI 工具。

# mongodb-org-shell

单独安装 mongodb-org-shell

``` bash
sudo apt install mongodb-org-shell
```