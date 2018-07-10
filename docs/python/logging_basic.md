> [Logging HOWTO](https://docs.python.org/3/howto/logging.html)

# 基础日志教程

日志记录是一种跟踪某些软件运行时发生的事件的方法。该软件的开发人员将日志记录调用添加到其代码中，以指示已发生某些事件。事件由描述性消息描述，该消息可以可选地包含可变数据（即每次事件发生时可能不同的数据）。事件也具有开发人员对事件的重要性；重要性也可以称为水平或严重程度。

## 何时使用日志

Logging 为简单的日志记录使用提供了一组便利功能。它们是 `debug()`, `info()`, `warning()`, `error()` 和 `critical()`。要确定何时使用日志记录，请参阅下表，其中列出了针对一组常见任务中的每个任务的最佳工具。

| 您要执行的任务 | 这项任务的最佳工具 |
|:--------------------------------
| 显示控制台输出，以便正常使用命令行脚本或程序 | `print()`
| 报告在程序正常运行期间发生的事件（例如，用于状态监测或故障调查） | `logging.info()`（或者 `logging.debug()` 用于非常详细的输出以用于诊断目的）
| 发出有关特定运行时事件的警告 | `warnings.warn()`： 在代码库中，如果问题是可以避免的，则应修改客户端应用程序以消除警告 <br><br> `logging.warning()`： 如果客户端应用程序无法处理该情况，但仍应注意该事件
| 报告有关特定运行时事件的错误 | 抛出异常 |
| 报告在不引发异常的情况下抑制错误（例如，长时间运行的服务器进程中的错误处理程序）| `logging.error()`, `logging.exception()` 或 `logging.critical()` 适用于特定错误和应用程序域

日志函数以它们用于跟踪的事件的级别或严重性命名。标准级别及其适用性描述如下（按严重程度递增）：

| 级别 | 什么时候使用 |
|:-------------------
| `DEBUG` | 详细信息，通常仅在诊断问题时才有意义。
| `INFO` | 确认事情按预期工作。
| `WARNING` | 表明发生了意外情况，或表明在不久的将来出现了一些问题（例如 “磁盘空间不足”）。但是该软件仍在按预期工作。
| `ERROR` | 由于更严重的问题，该软件无法执行某些功能。
| `CRITICAL` | 严重错误，表明程序本身可能无法继续运行。

默认级别为 `WARNING` ，这意味着将仅跟踪此级别及更高级别的事件，除非日志包已配置为执行其他操作。

可以以不同方式处理被跟踪的事件。处理跟踪事件的最简单方法是将它们打印到控制台。另一种常见方法是将它们写入磁盘文件。

## 一个简单的例子

一个非常简单的例子：

``` python
import logging
logging.warning('Watch out!')  # 将打印消息到控制台
logging.info('I told you so')  # 不会打印任何东西
```

如果您在脚本中输入这几行并运行它，您将看到：

```
WARNING:root:Watch out!
```

打印在控制台上。 `INFO` 消息不会出现，因为默认级别为 `WARNING` 。打印的消息包括记录调用中提供的事件的级别和描述的指示，即 “Watch out!”。暂时不要担心 'root' 部分：它将在后面解释。如果需要，可以非常灵活地格式化实际输出；格式化选项也将在稍后解释。

## 记录到文件

一种非常常见的情况是在文件中记录日志事件，所以让我们看看下一步。请务必在新启动的 Python 解释器中尝试以下操作，并且不要只继续上述会话：

``` python
import logging
logging.basicConfig(filename='example.log',level=logging.DEBUG)
logging.debug('This message should go to the log file')
logging.info('So should this')
logging.warning('And this, too')
```

现在，如果我们打开文件并查看我们的内容，我们应该找到日志消息：

```
DEBUG:root:This message should go to the log file
INFO:root:So should this
WARNING:root:And this, too
```

此示例还说明了如何设置作为跟踪阈值的日志记录级别。在这种情况下，因为我们将阈值设置为 `DEBUG` ，所以打印了所有消息。

如果要从命令行选项设置日志记录级别，例如：

```
--log=INFO
```

并且你有一个变量 `loglevel` 为 `--log` 传递的参数的值，你可以使用：

``` python
getattr(logging, loglevel.upper())
```

通过 `loglevel` 参数获取您将传递给 `basicConfig()` 的值。您可能还希望检查用户的输入值，如下例所示：

``` python
# 假设 loglevel 是从命令行参数中获取的字符串值。 转换为大写以允许用户
# 指定 --log=DEBUG 或 --log=debug
numeric_level = getattr(logging, loglevel.upper(), None)
if not isinstance(numeric_level, int):
    raise ValueError('Invalid log level: %s' % loglevel)
logging.basicConfig(level=numeric_level, ...)
```

对 `basicConfig()` 的调用应该在调用 `debug()`，`info()` 等之前进行。由于它是一次性的简单配置工具，只有第一次调用才会真正做事情：后续调用实际上是无效的。

如果多次运行上述脚本，则连续运行的消息将附加到文件 `example.log` 中。如果您希望每次运行重新开始，而不记住早期运行的消息，则可以指定 `filemode` 参数，通过将上例中的调用更改为：

``` python
logging.basicConfig(filename='example.log', filemode='w', level=logging.DEBUG)
```

输出将与之前相同，但不再附加日志文件，因此早期运行的消息将丢失。

## 多个模块中的日志记录

如果您的程序包含多个模块，这里有一个如何组织日志记录的示例：

``` python
# myapp.py
import logging
import mylib

def main():
    logging.basicConfig(filename='myapp.log', level=logging.INFO)
    logging.info('Started')
    mylib.do_something()
    logging.info('Finished')

if __name__ == '__main__':
    main()
```

``` python
# mylib.py
import logging

def do_something():
    logging.info('Doing something')
```

如果你运行 `myapp.py`，你应该在 `myapp.log` 中看到这个：

```
INFO:root:Started
INFO:root:Doing something
INFO:root:Finished
```

希望你们能看到。您可以使用 `mylib.py` 中的模式将此概括为多个模块。请注意，对于这种简单的使用模式，除了查看事件描述之外，仅仅通过查看日志文件，您不会知道您的消息来自应用程序中的何处。如果要跟踪消息的位置，则需要参考教程级别之外的文档 -- 请参阅[高级日志教程](#高级日志教程)。

## 记录变量数据

要记录变量数据，请使用格式字符串作为事件描述消息，并将变量数据作为参数附加。例如：

``` python
import logging
logging.warning('%s before you %s', 'Look', 'leap!')
```

将显示：

```
WARNING:root:Look before you leap!
```

如您所见，将可变数据合并到事件描述消息中使用旧的 `%` 样式字符串格式。这是为了向后兼容：日志包也支持更新的格式化选项，如 `str.format()` 和 `string.Template`。但探索它们超出了本教程的范围，相关信息请参阅 -- [在整个应用程序中使用特定格式样式](https://docs.python.org/3/howto/logging-cookbook.html#formatting-styles)

## 更改显示消息的格式

要更改用于显示消息的格式，您需要指定要使用的格式：

``` python
import logging
logging.basicConfig(format='%(levelname)s:%(message)s', level=logging.DEBUG)
logging.debug('This message should appear on the console')
logging.info('So should this')
logging.warning('And this, too')
```

这会打印：

```
DEBUG:This message should appear on the console
INFO:So should this
WARNING:And this, too
```

请注意，前面示例中出现的 “root” 已消失。对于可以出现在格式字符串中的一整套内容，你可以参考 [LogRecord 属性](https://docs.python.org/3/library/logging.html#logrecord-attributes)的文档，但为了简单使用，您只需要 `levelname`（重要性），`message`（事件描述，包括可变数据），并可能显示事件发生的时间。这将在下一节中介绍。

## 在消息中显示日期/时间

要显示事件的日期和时间，您可以在格式字符串中放置 `%(asctime)s`：

``` python
import logging
logging.basicConfig(format='%(asctime)s %(message)s')
logging.warning('is when this event was logged.')
```

应该打印这样的东西：

```
2010-12-12 11:41:42,612 is when this event was logged.
```

日期/时间显示的默认格式（如上所示）类似于 ISO8601 或 [RFC 3339](https://tools.ietf.org/html/rfc3339.html)。如果您需要更多地控制日期/时间的格式，请为 `basicConfig` 提供 `datefmt` 参数，如下例所示：

``` python
import logging
logging.basicConfig(format='%(asctime)s %(message)s', datefmt='%m/%d/%Y %I:%M:%S %p')
logging.warning('is when this event was logged.')
```

这会显示如下：

```
12/12/2010 11:46:36 AM is when this event was logged.
```

`datefmt` 参数的格式与 [`time.strftime()`](https://docs.python.org/3/library/time.html#time.strftime) 支持的格式相同。

## 下一步

基本教程到此结束。它应该足以让您启动并运行 logging。日志包提供了更多功能，但为了充分利用它，您需要花费更多的时间来阅读以下部分。如果你准备好了，可以拿一些你最喜欢的饮料继续。

如果您的日志记录需求很简单，那么使用上面的示例将日志记录合并到您自己的脚本中基本就可以了。

还在？您可以继续阅读接下来的几个部分，这些部分提供了比上面基本部分更高级/深入的教程。之后，您可以查看 [Logging Cookbook](https://docs.python.org/3/howto/logging-cookbook.html#logging-cookbook)。

# 高级日志教程

日志库采用模块化方法，并提供几类组件：记录器 (loggers)，处理器 (handlers)，过滤器 (filters) 和格式化器 (formatters)。

* 记录器公开应用程序代码直接使用的接口。
* 处理器将日志（由记录器创建）发送到适当的目标。
* 过滤器提供了更精细的设施，用于确定要输出的日志记录。
* 格式化器指定最终输出中的日志记录的布局。