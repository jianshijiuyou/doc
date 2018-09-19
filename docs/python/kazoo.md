> [原文链接](https://kazoo.readthedocs.io/en/latest/index.html)

# 安装

``` bash
pip install kazoo
```

# 基本使用

## 连接处理

要使用 Kazoo，必须创建一个 `KazooClient` 对象并建立连接：

``` python
from kazoo.client import KazooClient

zk = KazooClient(hosts='127.0.0.1:2181')
zk.start()
```

默认情况下，客户端将连接到本地默认端口（2181）上的 Zookeeper 服务器。您应该确保 Zookeeper 实际上是在那里运行，或者 `start` 命令将一直等到它的默认超时。

连接后，无论间歇性连接丢失或 Zookeeper 会话到期，客户端都将尝试保持连接。可以通过调用 `stop` 来指示客户端删除连接：

``` python
zk.stop()
```

### Logging 设置

如果没有为您的应用程序设置日志记录，您可能收到以下消息：

```
No handlers could be found for logger "kazoo.client"
```

要避免此问题，您至少可以执行以下操作：

``` python
import logging
from kazoo.client import KazooClient

console_handler = logging.StreamHandler()
console_handler.setLevel(logging.INFO)
logging.basicConfig(level=logging.INFO, handlers=[console_handler])

zk = KazooClient(hosts='127.0.0.1:2181')
zk.start()
```

### 收听连接事件

了解连接何时被删除，恢复或 Zookeeper 会话何时到期可能很有用。为了简化此过程，Kazoo 使用状态系统，并允许您注册在状态更改时调用的侦听器函数。


``` python
from kazoo.client import KazooState

def my_listener(state):
    if state == KazooState.LOST:
        # Register somewhere that the session was lost
    elif state == KazooState.SUSPENDED:
        # Handle being disconnected from Zookeeper
    else:
        # Handle being connected/reconnected to Zookeeper

zk.add_listener(my_listener)
```

### 了解 Kazoo 状态

`KazooState` 对象表示客户端转换的几个状态。始终可以通过查看 `state` 属性来确定客户端的当前状态。可能的状态是：

* LOST
* CONNECTED
* SUSPENDED

首次创建 `KazooClient` 实例时，它处于 `LOST` 状态。建立连接后，它将转换为 `CONNECTED` 状态。如果出现任何连接问题或者需要连接到不同的 Zookeeper 群集节点，它将转换为 `SUSPENDED` 以通知您当前无法运行命令。如果 Zookeeper 节点不再是仲裁的一部分，则连接也将丢失，从而导致 `SUSPENDED` 状态。

?> 应使用前面所述的侦听器监视这些状态，以便客户端根据连接状态正常运行。

当连接转换为 `SUSPENDED` 时，如果客户端正在执行需要与其他系统协商的操作（例如，使用锁定配方(Lock recipe)），则应暂停其正在执行的操作。重新建立连接后，客户端可以继续，具体取决于状态是否为 `LOST` 或再次直接转换为 `CONNECTED` 。

当连接转换为 `LOST` 时，Zookeeper 将删除已创建的任何临时节点(ephemeral node)。这会影响创建临时节点的所有配方(recipe)，例如锁定配方(Lock recipe)。在状态再次转换为 `CONNECTED` 后，需要重新获取 Lock。当会话过期或停止客户端连接时，会发生此转换。

**有效的状态转换**

* `LOST` -> `CONNECTED`

新连接，或之前丢失的连接。

* `CONNECTED` -> `SUSPENDED`

连接上发生服务器连接丢失。

* `CONNECTED` -> `LOST`

仅在建立连接后提供无效的身份验证凭据时才会发生。

* `SUSPENDED` -> `LOST`

连接恢复到服务器，但随着会话过期而丢失。

* `SUSPENDED` -> `CONNECTED`

丢失的连接已恢复。

### 只读连接

Zookeeper 3.4 及更高版本支持只读模式。必须为 Zookeeper 群集中的服务器启用此模式，以便客户端使用它。要在 Kazoo 中使用此模式，应该在 `read_only` 选项设置为 `True` 的情况下调用 `KazooClient` 。这将让客户端连接到已经变为只读的 Zookeeper 节点，客户端将继续扫描其他读写节点。

``` python
from kazoo.client import KazooClient

zk = KazooClient(hosts='127.0.0.1:2181', read_only=True)
zk.start()
```

添加了 `KeeperState` 的新属性 `CONNECTED_RO` 。上面的连接状态仍然有效，但是在 `CONNECTED` 时，您需要检查客户端非简化状态以查看连接是否为 `CONNECTED_RO` 。举个栗子。

``` python
from kazoo.client import KazooState
from kazoo.client import KeeperState

@zk.add_listener
def watch_for_ro(state):
    if state == KazooState.CONNECTED:
        if zk.client_state == KeeperState.CONNECTED_RO:
            print("Read only mode!")
        else:
            print("Read/Write mode!")
```

重要的是要注意将 `KazooState` 传递给侦听器，但只有通过将非简化客户端状态与 `KeeperState` 对象进行比较才能获得只读信息。

!> 使用只读模式的客户端不应使用任何配方(recipe)。

## Zookeeper CRUD

Zookeeper 包含几个用于创建，读取，更新和删除 Zookeeper 节点的功能（此处称为 `znode` 或 `node` ）。 Kazoo 添加了几种便捷方法和更多 Pythonic API。

### 创建 Node

方法：

* `ensure_path()`
* `create()`

`ensure_path()` 将递归地创建节点和路径中所需路径中的任何节点，但不能为节点设置数据，只能设置 ACL。

`create()` 创建一个节点，可以设置节点上的数据以及 watch 函数。它需要首先存在路径，除非 `makepath` 选项设置为 `True` 。

``` python
# 确保路径，必要时创建
zk.ensure_path("/my/favorite")

# 创建节点和数据
zk.create("/my/favorite/node", b"a value")
```

### 读取数据

方法：

* `exists()`
* `get()`
* `get_children()`

`exists()` 检查节点是否存在。

`get()` 获取节点的数据以及 `ZnodeStat` 结构中的详细节点信息。

`get_children()` 获取给定节点的子节点列表。

``` python
# 确定节点是否存在
if zk.exists("/my/favorite"):
    # Do something

# 打印节点及其数据的版本
data, stat = zk.get("/my/favorite")
print("Version: %s, data: %s" % (stat.version, data.decode("utf-8")))

# 列出子节点
children = zk.get_children("/my/favorite")
print("There are %s children with names %s" % (len(children), children))
```

### 更新数据

方法：

* `set()`

`set()` 更新给定节点的数据。可以提供节点的版本，在更新数据之前需要匹配，或者将引发 `BadVersionError` 而不是更新。

``` python
zk.set("/my/favorite", b"some data")
```

### 删除节点

方法：

* `delete()`

`delete()` 删除节点，也可以选择递归删除节点的所有子节点。删除节点时可以提供版本，该节点在删除节点之前需要与节点版本匹配，否则将引发 `BadVersionError` 而不是删除。

``` python
# recursive 递归删除所有子节点
zk.delete("/my/favorite/node", recursive=True)
```

## 重试命令

如果 Zookeeper 服务器出现故障或无法访问，则与 Zookeeper 的连接可能会中断。默认情况下，kazoo 不会重试命令，因此这些失败将导致引发异常。为了帮助解决故障，kazoo 附带了一个 `retry()` 帮助程序，如果其中一个 Zookeeper 连接异常被引发，它将重试一个函数。

举个栗子

``` python
result = zk.retry(zk.get, "/path/to/node")
```

某些命令可能具有唯一的行为，不保证基于每个命令的自动重试。例如，如果创建节点，则在命令成功返回但实际创建节点之前，连接可能会丢失。这会导致再次运行时引发 `kazoo.exceptions.NodeExistsError` 。当使用 `ephemeral` 和 `sequence` 选项集创建节点时会出现类似的独特情况，这在 Zookeeper 站点上有说明。

由于 `retry()` 方法接受函数调用及其参数，因此可以将运行多个 Zookeeper 命令的函数传递给它，以便在连接丢失时重试整个函数。

来自锁实现的这个片段显示了它如何使用重试来重新运行获取锁的函数，并检查它是否已经创建以处理这种情况：

``` python
# kazoo.recipe.lock snippet

def acquire(self):
    """Acquire the mutex, blocking until it is obtained"""
    try:
        self.client.retry(self._inner_acquire)
        self.is_acquired = True
    except KazooException:
        # if we did ultimately fail, attempt to clean up
        self._best_effort_cleanup()
        self.cancelled = False
        raise

def _inner_acquire(self):
    self.wake_event.clear()

    # make sure our election parent node exists
    if not self.assured_path:
        self.client.ensure_path(self.path)

    node = None
    if self.create_tried:
        node = self._find_node()
    else:
        self.create_tried = True

    if not node:
        node = self.client.create(self.create_path, self.data,
            ephemeral=True, sequence=True)
        # strip off path to node
        node = node[len(self.path) + 1:]
```

`create_tried` 记录在返回节点名称之前连接丢失的情况下是否已尝试创建节点。

### 自定义重试

有时，您可能希望对与 `retry()` 方法不同的命令或命令集具有特定的重试策略。您可以使用您喜欢的特定重试策略手动创建 `KazooRetry` 实例：

``` python
from kazoo.retry import KazooRetry

kr = KazooRetry(max_tries=3, ignore_expire=False)
result = kr(client.get, "/some/path")
```

这将最多重试 `client.get` 命令 `3` 次，并在会话到期时抛出异常。您还可以使用在重试期间忽略会话到期的默认行为创建实例。

## 监控 Node

Kazoo 可以在节点上设置监视函数，可以在节点发生变化时或节点的子节点发生变化时触发。对节点或子节点的此更改也可以是要删除的节点或其子节点。

可以通过两种不同的方式设置观察者，第一种是 Zookeeper 默认支持的一次性观看事件的风格。这些监视函数将由 kazoo 调用一次，并且不会接收会话事件，这与本机 Zookeeper 监视器不同。使用此风格需要将 watch 函数传递给以下方法之一：

* `get()`
* `get_children()`
* `exists()`

当节点上的数据发生更改或节点本身被删除时，将调用传递给 `get()` 或 `exists()` 的监视函数。它将传递一个 `WatchedEvent` 实例。

``` python
def my_func(event):
    # check to see what the children are now

# Call my_func when the children change
children = zk.get_children("/my/favorite/node", watch=my_func)
```

Kazoo 包含一个更高级别的 API，可以监视数据和子代修改，因为它不需要在每次触发事件时重新设置 watch。它还会在观看节点或子节点时观察节点子节点时传入数据和 `ZnodeStat` 。使用此 API 注册的监视函数将在每次更改时立即调用，或者直到函数返回 `False` 。如果 `allow_session_lost` 设置为 `True` ，则会话丢失时将不再调用该函数。

以下方法提供此功能：

* `ChildrenWatch`
* `DataWatch`

这些类可直接在 `KazooClient` 实例上使用，并且在以这种方式使用时不需要传入客户端对象。通过实例化任一类返回的实例可以直接调用，允许它们用作装饰器：

``` python
@zk.ChildrenWatch("/my/favorite/node")
def watch_children(children):
    print("Children are now: %s" % children)
# Above function called immediately, and from then on

@zk.DataWatch("/my/favorite")
def watch_node(data, stat):
    print("Version: %s, data: %s" % (stat.version, data.decode("utf-8")))
```

## 事务

Zookeeper 3.4 及更高版本支持一次发送多个命令，这些命令将作为单个原子单元提交。要么他们都会成功，要么他们都会失败。事务的结果将是事务中每个命令的成功/失败结果的列表。

``` python
transaction = zk.transaction()
transaction.check('/node/a', version=3)
transaction.create('/node/b', b"a value")
results = transaction.commit()
```

`transaction()` 方法返回 `TransactionRequest` 实例。可以调用它的方法来排队要在事务中完成的命令。当事务准备好发送时，调用它上面的 `commit()` 方法。

在上面的示例中，除非正在使用事务，否则有一个命令不可用，请检查。这可以检查特定版本的节点，如果节点与它应该处于的版本不匹配，则会使该节点的事务失败。在这种情况下，节点 `/node/a` 必须是版本 `3` 或 `/node/b` 不会被创建。
