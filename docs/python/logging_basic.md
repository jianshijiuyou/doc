> [Logging HOWTO](https://docs.python.org/3/howto/logging.html)

# 基础日志教程

日志记录是一种跟踪某些软件运行时发生的事件的方法。该软件的开发人员将日志记录调用添加到其代码中，以指示已发生某些事件。事件由描述性消息描述，该消息可以可选地包含可变数据（即每次事件发生时可能不同的数据）。事件也具有开发人员对事件的重要性；重要性也可以称为水平或严重程度。

## 何时使用日志

Logging 为简单的日志记录使用提供了一组便利功能。它们是 `debug()`, `info()`, `warning()`, `error()` 和 `critical()`。要确定何时使用日志记录，请参阅下表，其中列出了针对一组常见任务中的每个任务的最佳工具。

| 您要执行的任务 | 这项任务的最佳工具 |
|:-------------|:-------------------
| 显示控制台输出，以便正常使用命令行脚本或程序 | `print()`
| 报告在程序正常运行期间发生的事件（例如，用于状态监测或故障调查） | `logging.info()`（或者 `logging.debug()` 用于非常详细的输出以用于诊断目的）
| 发出有关特定运行时事件的警告 | `warnings.warn()`： 在代码库中，如果问题是可以避免的，则应修改客户端应用程序以消除警告 <br><br> `logging.warning()`： 如果客户端应用程序无法处理该情况，但仍应注意该事件
| 报告有关特定运行时事件的错误 | 抛出异常 |
| 报告在不引发异常的情况下抑制错误（例如，长时间运行的服务器进程中的错误处理程序）| `logging.error()`, `logging.exception()` 或 `logging.critical()` 适用于特定错误和应用程序域

日志函数以它们用于跟踪的事件的级别或严重性命名。标准级别及其适用性描述如下（按严重程度递增）：

| 级别 | 什么时候使用 |
|:-----|:--------------
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

日志事件信息在 [LogRecord](https://docs.python.org/3/library/logging.html#logging.LogRecord) 实例中的记录器，处理器，过滤器和格式化器之间传递。

通过在 `Logger` 类的实例（以下称为记录器）上调用方法来执行日志记录。每个实例都有一个名称，它们在概念上以点（句点）作为分隔符排列在命名空间层次结构中。例如，名为 “scan” 的记录器是记录器 'scan.text'，'scan.html' 和 'scan.pdf' 的父级。记录器名称可以是您想要的任何名称，并指明记录消息来源的应用程序区域。

在命名记录器时使用的一个好习惯是在每个使用日志记录的模块中使用模块级记录器，命名如下：

``` python
logger = logging.getLogger(__name__)
```

这意味着记录器名称跟踪包/模块层次结构，并且直观地显示从记录器名称记录事件的位置。

记录器层次结构的根称为根记录器。这是函数 `debug()`，`info()`，`warning()`，`error()` 和 `critical()` 使用的记录器，它只调用根记录器的同名方法。函数和方法具有相同的签名。根记录器的名称在记录的输出中打印为 “root”。

当然，可以将消息记录到不同的目的地。软件包中包含支持，用于将日志消息写入文件，HTTP GET/POST 位置，通过 SMTP 发送电子邮件，通用套接字，队列或特定于操作系统的日志记录机制（如 syslog 或 Windows NT 事件日志）。目标由处理器类提供。如果内置处理器类未满足你的特殊要求，则可以创建自己的日志目标类。

默认情况下，没有为任何日志记录消息设置目标。您可以使用 `basicConfig()` 指定目标（例如控制台或文件），如前文中所示。如果调用函数 `debug()`，`info()`，`warning()`，`error()` 和 `critical()`，它们将检查是否没有设置目标；如果未设置，则在委派给根记录器执行实际消息输出之前，他们将设置控制台的目标（`sys.stderr`）和显示消息的默认格式。

`basicConfig()` 为消息设置的默认格式为：

```
severity:logger name:message
```

您可以通过使用 `format` 关键字参数将格式字符串传递给 `basicConfig()` 来更改此设置。有关如何构造格式字符串的所有选项，请参阅[格式化对象](https://docs.python.org/3/library/logging.html#formatter-objects)。

## 记录流程

记录器和处理器中的日志事件信息流程如下图所示。

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/images/c3d88410-9d19-4c99-b9c8-752281662401.png)

## 记录器

`Logger` 对象有三重作业。首先，它们向应用程序代码公开了几种方法，以便应用程序可以在运行时记录消息。其次，记录器对象根据严重性（默认过滤工具）或过滤器对象确定要处理的日志消息。最后，记录器对象将相关的日志消息传递给所有感兴趣的日志处理器。

记录器对象上使用最广泛的方法分为两类：配置和消息发送。

这些是最常见的配置方法：

* `Logger.setLevel()` 指定记录器将处理的日志级别，其中 `debug` 是最低内置日志级别，`critical` 是最高内置日志级别。例如，如果日志级别为 INFO，则记录器将仅处理 INFO，WARNING，ERROR 和 CRITICAL 消息，并将忽略 DEBUG 消息。
* `Logger.addHandler()` 和 `Logger.removeHandler()` 从记录器对象中添加和删除处理器对象。
* `Logger.addFilter()` 和 `Logger.removeFilter()` 从记录器对象中添加和删除过滤器对象。

您不需要始终在您创建的每个记录器上调用这些方法。请参阅本节的最后两段。

配置 `logger` 对象后，以下方法将创建日志消息：

* `Logger.debug()`，`Logger.info()`，`Logger.warning()`，`Logger.error()` 和 `Logger.critical()` 都创建日志记录，其中包含一条消息和一个与其各自方法名称对应的级别。该消息实际上是一个格式字符串，可能包含 `%s`，`%d`，`%f` 的标准字符串替换语法，依此类推。其余参数是与消息中的替换字段对应的对象列表。关于 `**kwargs`，日志记录方法仅关注 `exc_info` 的关键字，并使用它来确定是否记录异常信息。
* `Logger.exception()` 创建类似于 `Logger.error()` 的日志消息。区别在于 `Logger.exception()` 与其一起转储堆栈跟踪。仅从异常处理程序调用此方法。
* `Logger.log()` 将日志级别作为显式参数。对于记录消息而言，这比使用上面列出的日志级别便捷方法要详细一些，但这可以自定义日志级别。

`getLogger()` 返回对具有指定名称的记录器实例的引用（如果已提供），如果不是则返回 `root`。名称是以句点分隔的层次结构。对具有相同名称的 `getLogger()` 的多次调用将返回对同一记录器对象的引用。在分层列表中较低的记录器是列表中较高的记录器的子项。例如，给定一个名为 `foo` 的记录器，名称为 `foo.bar`，`foo.bar.baz` 和 `foo.bam` 的记录器都是 `foo` 的后代。

记录器具有有效级别的概念。如果未在记录器上显式设置级别，则使用其父级别作为其有效级别。如果父级没有明确的级别设置，则再检查其父级，依此类推 - 搜索所有祖先，直到找到明确设置的级别。根记录器始终设置了显式级别（默认情况下为 WARNING）。在决定是否处理事件时，记录器的有效级别用于确定事件是否传递给记录器的处理器。

子记录器将消息传播到与其祖先记录器相关联的处理器。因此，不必为应用程序使用的所有记录器定义和配置处理器。为顶级记录器配置处理器并根据需要创建子记录器就足够了。（但是，您可以通过将记录器的 `propagate` 属性设置为 `False` 来关闭传播。）

## 处理器

处理器对象负责将适当的日志消息（基于日志消息的严重性）分派给处理器的指定目标。`Logger` 对象可以使用 `addHandler()` 方法向自身添加零个或多个处理器对象。作为示例场景，应用程序可能希望将所有日志消息发送到日志文件，将错误或更高的所有日志消息发送到标准输出，以及将至关重要的所有消息发送到电子邮箱。此方案需要三个单独的处理器，其中每个处理器负责将特定严重性的消息发送到特定位置。

标准库包含很多处理器类型（请参阅[常用处理器](https://docs.python.org/3/howto/logging.html#useful-handlers)）；这些教程在其示例中主要使用 `StreamHandler` 和 `FileHandler`。

处理器中很少有方法需要应用程序开发人员关注。与使用内置处理器对象（即不创建自定义处理器）的应用程序开发人员相关的唯一处理器方法是以下配置方法：

* 与记录器对象一样，`setLevel()` 方法指定将分派到适当目标的最低严重性。为什么有两个 `setLevel()` 方法？记录器中设置的级别确定将传递给其处理器的消息的严重性。而每个处理器中设置的级别确定处理器将发送哪些消息。
* `setFormatter()` 选择要使用的此处理器的 Formatter 对象。
* `addFilter()` 和 `removeFilter()` 分别在处理器上配置和取消配置过滤器对象。 

应用程序代码不应直接实例化和使用 `Handler` 的实例。相反，`Handler` 类是一个基类，它定义了所有处理程序应具有的接口，并建立了子类可以使用（或覆盖）的一些默认行为。

## 格式化器

`Formatter` 对象配置日志消息的最终顺序，结构和内容。与基本 `logging.Handler` 类不同，应用程序代码可以实例化 `formatter` 类，但如果应用程序需要特殊行为，则可能会对 `formatter` 进行子类化。构造函数有三个可选参数 - 消息格式字符串，日期格式字符串和样式指示符。

`logging.Formatter.__init__(fmt=None, datefmt=None, style='%')`

如果没有消息格式字符串，则默认使用原始消息。如果没有日期格式字符串，则默认日期格式为：

```
%Y-%m-%d %H:%M:%S
```

最后加上毫秒数。样式是 `%`，`'{'` 或 `'$'` 之一。如果未指定其中一个，则使用 `'%'`。

如果 `style` 为 `'%'`，则消息格式字符串使用 `%(<dictionary key>)s` 样式字符串替换；[LogRecord 属性](https://docs.python.org/3/library/logging.html#logrecord-attributes)中记录了可能的键。如果 `style` 为 “{”，则假定消息格式字符串与 `str.format()`（使用关键字参数）兼容，而如果 `style` 为 “$”，则消息格式字符串应符合 ` string.Template.substitute()` 的预期。

Python 3.2 中添加了 `style` 参数。

以下消息格式字符串将按以下顺序以人类可读的格式记录时间，消息的严重性和消息的内容：

```
'%(asctime)s - %(levelname)s - %(message)s'
```

Formatters 使用用户可配置的函数将记录的创建时间转换为元组。默认情况下，使用 `time.localtime()`；要为特定格式化器实例更改此值，请将实例的 `converter` 属性设置为与 `time.localtime()` 或 `time.gmtime()` 具有相同签名的函数。要为所有格式化程序更改它，例如，如果要在 GMT 中显示所有记录时间，请在 Formatter 类中设置 `converter` 属性（用 GMT 则显示 `time.gmtime`）。

## 配置日志记录

程序员可以通过三种方式配置日志记录：

1. 调用上面列出的配置方法显式创建记录器，处理器和格式化器。
2. 创建日志配置文件并使用 `fileConfig()` 函数读取它。
3. 创建配置信息字典并将其传递给 `dictConfig()` 函数。

有关最后两个选项的参考文档，请参阅 [配置函数](https://docs.python.org/3/library/logging.config.html#logging-config-api)。以下示例使用 Python 代码配置一个非常简单的记录器，一个控制台处理器和一个简单的格式化器：

``` python
import logging

# create logger
logger = logging.getLogger('simple_example')
logger.setLevel(logging.DEBUG)

# 创建控制台处理器并设置级别进行调试
ch = logging.StreamHandler()
ch.setLevel(logging.DEBUG)

# create formatter
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')

# add formatter to ch
ch.setFormatter(formatter)

# add ch to logger
logger.addHandler(ch)

# 'application' code
logger.debug('debug message')
logger.info('info message')
logger.warn('warn message')
logger.error('error message')
logger.critical('critical message')
```

从命令行运行此模块将生成以下输出：

``` python
$ python simple_logging_module.py
2005-03-19 15:10:26,618 - simple_example - DEBUG - debug message
2005-03-19 15:10:26,620 - simple_example - INFO - info message
2005-03-19 15:10:26,695 - simple_example - WARNING - warn message
2005-03-19 15:10:26,697 - simple_example - ERROR - error message
2005-03-19 15:10:26,773 - simple_example - CRITICAL - critical message
```

以下 Python 模块创建的记录器，处理器和格式化器与上面列出的示例几乎完全相同，唯一的区别是对象的名称：

``` python
import logging
import logging.config

logging.config.fileConfig('logging.conf')

# create logger
logger = logging.getLogger('simpleExample')

# 'application' code
logger.debug('debug message')
logger.info('info message')
logger.warn('warn message')
logger.error('error message')
logger.critical('critical message')
```

这是 `logging.conf` 文件：

```
[loggers]
keys=root,simpleExample

[handlers]
keys=consoleHandler

[formatters]
keys=simpleFormatter

[logger_root]
level=DEBUG
handlers=consoleHandler

[logger_simpleExample]
level=DEBUG
handlers=consoleHandler
qualname=simpleExample
propagate=0

[handler_consoleHandler]
class=StreamHandler
level=DEBUG
formatter=simpleFormatter
args=(sys.stdout,)

[formatter_simpleFormatter]
format=%(asctime)s - %(name)s - %(levelname)s - %(message)s
datefmt=
```

输出几乎与基于非配置文件的示例相同：

```
$ python simple_logging_config.py
2005-03-19 15:38:55,977 - simpleExample - DEBUG - debug message
2005-03-19 15:38:55,979 - simpleExample - INFO - info message
2005-03-19 15:38:56,054 - simpleExample - WARNING - warn message
2005-03-19 15:38:56,055 - simpleExample - ERROR - error message
2005-03-19 15:38:56,130 - simpleExample - CRITICAL - critical message
```

您可以看到配置文件方法与 Python 代码方法相比具有一些优势，主要是配置和代码的分离以及非编码器轻松修改日志记录属性的能力。

!> 警告：`fileConfig()` 函数采用默认参数 `disable_existing_loggers` ，出于向后兼容的原因，默认参数为 `True`。这可能是您想要的，也可能不是，因为它会导致在 `fileConfig()` 调用之前存在的任何记录器被禁用，除非它们（或祖先）在配置中明确命名。有关详细信息，请参阅参考文档，如果需要，请为此参数指定 `False`。 <br><br>传递给`dictConfig()` 的字典也可以使用键 `disable_existing_loggers` 指定一个布尔值，如果未在字典中明确指定，则默认情况下将其解释为 `True`。这会导致上面描述的记录器禁用行为，这可能不是您想要的 - 在这种情况下，请显式提供值为 `False` 的键。

请注意，配置文件中引用的类名称需要相对于日志记录模块，或者可以使用常规导入机制解析的绝对值。因此，您可以使用 `WatchedFileHandler`（相对于日志记录模块）或 `mypackage.mymodule.MyHandler`（对于在 `mypackage` 包和模块 `mymodule` 中定义的类，其中 `mypackage` 在 Python 导入路径上可用）

在 Python 3.2 中，引入了一种新的配置日志记录的方法，使用字典来保存配置信息。这提供了上面概述的基于配置文件的方法的功能的超集，并且是新应用程序和部署的推荐配置方法。因为 Python 字典用于保存配置信息，并且由于您可以使用不同的方式填充该字典，因此您有更多的配置选项。例如，您可以使用 JSON 格式的配置文件，或者，如果您有权访问 YAML 处理函数，则可以使用 YAML 格式的文件来填充配置字典。或者，当然，您可以在 Python 代码中构建字典，通过套接字以序列化形式接收它，或者使用对您的应用程序有意义的任何方法。

以下是与上述相同配置的示例，采用YAML格式，用于新的基于字典的方法：

``` python
version: 1
formatters:
  simple:
    format: '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
handlers:
  console:
    class: logging.StreamHandler
    level: DEBUG
    formatter: simple
    stream: ext://sys.stdout
loggers:
  simpleExample:
    level: DEBUG
    handlers: [console]
    propagate: no
root:
  level: DEBUG
  handlers: [console]
```

有关使用字典进行日志记录的详细信息，请参阅 [配置函数](https://docs.python.org/3/library/logging.config.html#logging-config-api)。

## 如果没有提供配置会发生什么

如果未提供日志记录配置，则可能出现需要输出日志记录事件但无法找到输出事件的处理器的情况。在这些情况下，日志包的行为取决于 Python 版本。

对于 3.2 之前的 Python 版本，行为如下：

* 如果 `logging.raiseExceptions` 为 `False`（生产模式），则会以静默方式删除该事件。
* 如果 `logging.raiseExceptions` 为 `True`（开发模式），则会打印一条消息 “无法找到记录器 `X.Y.Z` 的处理器”。

在 Python 3.2 及更高版本中，行为如下：

* 该事件使用 “最后的处理器” 输出，存储在 `logging.lastResort` 中。此内部处理器不与任何记录器关联，并且像 `StreamHandler` 一样将事件描述消息写入 `sys.stderr` 的当前值（因此尊重可能有效的任何重定向）。没有对消息进行格式化 - 只打印裸事件描述消息。处理程序的级别设置为 `WARNING`，因此将输出此级别和更高级别的所有事件。

要获取 3.2 之前的行为，`logging.lastResort` 可以设置为 `None`。

## 为库配置日志记录

在开发使用日志记录的库时，您应该注意记录库如何使用日志记录 - 例如，使用的记录器的名称。还需要考虑其日志记录配置。如果应用程序不使用日志记录，并且库代码进行日志记录调用，则（如上一节所述）严重性为 `WARNING` 和更高的事件将打印到 `sys.stderr`。这被认为是最好的默认行为。

如果由于某种原因您不希望在没有任何日志记录配置的情况下打印这些消息，则可以将无操作处理器附加到库的顶级记录器。这样可以避免打印消息，因为将始终为库的事件找到处理器：它不会产生任何输出。如果库用户配置日志以供应用程序使用，可能是配置将添加一些处理器，如果级别已适当配置，则在库代码中进行的日志记录调用将正常地将输出发送给这些处理器。

日志包中包含一个什么都不做的处理器：`NullHandler`（自 Python 3.1 起）。可以将此处理程序的实例添加到库使用的日志记录命名空间的顶级记录器中（如果要在没有日志记录配置的情况下阻止将库的记录事件输出到 `sys.stderr`）。如果库 `foo` 的所有日志记录都是使用名称匹配 `'foo.x'`，`'foo.x.y'` 等的记录器完成的，那么代码：

``` python
import logging
logging.getLogger('foo').addHandler(logging.NullHandler())
```

应该有所期望的效果。如果组织生成了许多库，则指定的记录器名称可以是 “orgname.foo” 而不仅仅是 “foo”。

?> 注意：强烈建议您不要将 `NullHandler` 以外的任何处理程序添加到库的记录器中。这是因为处理器的配置是使用您的库的应用程序开发人员的特权。用程序开发人员了解他们的目标受众以及哪些处理器最适合他们的应用程序：如果你在引擎盖下添加处理器，你可能会干扰他们执行单元测试和提供符合他们要求的日志的能力。

# 记录级别

日志记录级别的数值在下表中给出。如果要定义自己的级别，并且需要它们具有相对于预定义级别的特定值，则主要关注这些级别。如果您使用相同的数值定义级别，它将覆盖预定义的值；预定义的名称将丢失。

| 级别 | 数值 |
|:-----|:--------
| `CRITICAL` | 50 |
| `ERROR` | 40 |
| `WARNING` | 30 |
| `INFO` | 20 |
| `DEBUG` | 10 |
| `NOTSET` | 0 |

级别也可以与记录器相关联，由开发人员或通过加载已保存的日志记录配置来设置。在记录器上调用日志记录方法时，记录器会将其自身级别与方法调用关联的级别进行比较。如果记录器的级别高于方法调用的级别，则实际上不会生成任何记录消息。这是控制日志输出详细程度的基本机制。

记录消息被编码为 `LogRecord` 类的实例。当记录器决定实际记录事件时，将从记录消息创建 `LogRecord` 实例。

记录消息通过使用处理器来处理调度机制，处理器是 `Handler` 类的子类的实例。处理器负责确保记录的消息（以 `LogRecord` 的形式）最终位于特定位置（或一组位置），这对于该消息的目标受众是有用的（例如最终用户，支持服务台员工，系统管理员，开发人员）。处理器传递用于特定目标的 `LogRecord` 实例。每个记录器可以有零个，一个或多个与之关联的处理器（通过 `Logger` 的 `addHandler()` 方法）。除了与记录器直接关联的任何处理器之外，还会调用与记录器的所有祖先关联的所有处理器来分派消息（除非记录器的 `propagate` 标志设置为 `False` 值，此时传递给祖先处理程序停止）。

就像记录器一样，处理器可以具有与它们相关联的级别。处理器的级别充当过滤器，其方式与记录器级别相同。如果处理器决定实际调度事件，则使用 `emit()` 方法将消息发送到其目标。大多数用户定义的 `Handler` 子类都需要覆盖此 `emit()`。

## 自定义级别

定义您自己的级别是可以的，但不一定是必要的，因为现有级别是根据实践经验选择的。但是，如果您确信需要自定义级别，则在执行此操作时应特别小心，如果您正在开发库，则定义自定义级别可能是一个非常糟糕的主意。那是因为如果多个库作者都定义了他们自己的自定义级别，由于给定的数值对于不同的库而言可能意味着不同的事物，因此有可能使用这些多个库的日志输出对于使用开发者来说难以控制和（或）解释。

# 常用处理器

除了基本的 `Handler` 类之外，还提供了许多有用的子类：

1. `StreamHandler` 实例将消息发送到流（类文件对象）。
2. `FileHandler` 实例将消息发送到磁盘文件。
3. `BaseRotatingHandler` 是在某个点切割日志文件的处理器的基类。它并不意味着直接实例化。而是使用 `RotatingFileHandler` 或 `TimedRotatingFileHandler`。
4. `RotatingFileHandler` 实例将消息发送到磁盘文件，支持最大日志文件大小和日志文件切割。
5. `TimedRotatingFileHandler` 实例将消息发送到磁盘文件，以特定的时间间隔切割日志文件。
6. `SocketHandler` 实例将消息发送到 TCP/IP 套接字。从 3.4 开始，也支持 Unix 域套接字。
7. `DatagramHandler` 实例将消息发送到 UDP 套接字。从 3.4 开始，也支持 Unix 域套接字。
8. `SMTPHandler` 实例将消息发送到指定的电子邮件地址。
9. `SysLogHandler` 实例将消息发送到 Unix syslog 守护程序，可以是在远程计算机上。
10. `NTEventLogHandler` 实例将消息发送到 Windows NT/2000/XP 事件日志。
11. `MemoryHandler` 实例将消息发送到内存中的缓冲区，只要满足特定条件，就会刷新内存中的缓冲区。
12. `HTTPHandler` 实例使用 GET 或 POST 语义将消息发送到 HTTP 服务器。
13. `WatchedFileHandler` 实例监视他们要记录的文件。如果文件发生更改，则会关闭该文件并使用文件名重新打开。此处理程序仅在类 Unix 系统上有用; Windows 不支持使用的基础机制。
14. `QueueHandler` 实例将消息发送到队列，例如队列或多处理模块中实现的队列。
15. `NullHandler` 实例不会对错误消息执行任何操作。

`NullHandler` ，`StreamHandler` 和 `FileHandler` 类在核心日志包中定义。其他处理程序在子模块 `logging.handlers` 中定义。（还有另一个子模块 `logging.config`，用于配置功能。）

记录的消息被格式化以便通过 `Formatter` 类的实例进行呈现。它们使用适合与 `%` 运算符和字典一起使用的格式字符串进行初始化。

对于批量格式化多个消息，可以使用 `BufferingFormatter` 的实例。除了格式字符串（应用于批处理中的每个消息）之外，还提供了标题和尾部格式字符串。

当基于记录器级别和（或）处理器级别的过滤不够时，可以将过滤器的实例添加到 `Logger` 和 `Handler` 实例（通过他们的 `addFilter()` 方法）。在决定进一步处理消息之前，记录器和处理器都会查询其所有过滤器以获取权限。如果任何过滤器返回 false 值，则不会进一步处理该消息。

基本的过滤器功能允许按特定的记录器名称进行过滤。如果使用此功能，则允许通过过滤器发送到指定记录器及其子项的消息，并删除所有其他消息。

# 记录期间引发的异常

日志包旨在吞噬登录生产时发生的异常。这样可以在处理日志记录事件时发生错误 - 例如记录错误配置，网络或其他类似错误 - 不要导致使用日志记录的应用程序过早终止。

永远不会吞下 `SystemExit` 和 `KeyboardInterrupt` 异常。在 `Handler` 子类的 `emit()` 方法期间发生的其他异常将传递给其 `handleError()` 方法。

`Handler` 中 `handleError()` 的默认实现检查是否设置了模块级变量 `raiseExceptions`。如果设置，则会向 `sys.stderr` 打印回溯。如果未设置，则吞下异常。

?> 注意：`raiseExceptions` 的默认值为 `True`。这是因为在开发期间，您通常希望收到发生的任何异常的通知。建议您将生产使用的 `raiseExceptions` 设置为 `False` 。

# 使用任意对象作为消息

在前面的部分和示例中，假设记录事件时传递的消息是字符串。但是，这不是唯一的可能性。您可以将任意对象作为消息传递，并且当日志记录系统需要将其转换为字符串表示时，将调用其 `__str__()` 方法。实际上，如果您愿意，可以避免完全计算字符串表示 - 例如， `SocketHandler` 通过序列化并通过线路发送事件来发出事件。
