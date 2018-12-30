> 官方原文： https://docs.python.org/3/library/pathlib.html

# pathlib - 面向对象的 filesystem paths

?> Python 3.4 中的新功能

该模块提供表示文件系统路径的类，其语义适用于不同的操作系统。路径类在纯路径之间划分，纯路径提供纯粹的计算操作而没有 `I/O`，以及具体路径，它继承纯路径但也提供 `I/O` 操作。

![](http://pikachu666.oss-cn-hongkong.aliyuncs.com/images/pathlib-inheritance.png)

如果您之前从未使用过此模块，或者只是不确定哪个类适合您的任务，那么 Path 很可能就是您所需要的。

纯路径在某些特殊情况下很有用;例如：

1. 如果要在 Unix 计算机上操作 Windows 路径（反之亦然）。在 Unix 上运行时无法实例化 `WindowsPath` ，但可以实例化 `PureWindowsPath` 。
2. 您希望确保您的代码仅操作路径而不实际访问操作系统。在这种情况下，实例化其中一个纯类可能很有用，因为那些只是没有任何系统访问操作的类。

# 基本用法

导入主类：

``` python
>>> from pathlib import Path
```

列出子目录：

``` python
>>> p = Path('.')
>>> [x for x in p.iterdir() if x.is_dir()]
[PosixPath('.hg'), PosixPath('docs'), PosixPath('dist'),
 PosixPath('__pycache__'), PosixPath('build')]
```

在此目录树中列出 Python 源文件：

``` python
>>> list(p.glob('**/*.py'))
[PosixPath('test_pathlib.py'), PosixPath('setup.py'),
 PosixPath('pathlib.py'), PosixPath('docs/conf.py'),
 PosixPath('build/lib/pathlib.py')]
```

在目录树中导航：

``` python
>>> p = Path('/etc')
>>> q = p / 'init.d' / 'reboot'
>>> q
PosixPath('/etc/init.d/reboot')
>>> q.resolve()
PosixPath('/etc/rc.d/init.d/halt')
```

查询路径属性：

``` python
>>> q.exists()
True
>>> q.is_dir()
False
```

打开文件：

``` python
>>> with q.open() as f: f.readline()
...
'#!/bin/bash\n'
```

# 纯路径(Pure paths)

纯路径对象提供了实际上不访问文件系统的路径处理操作。有三种方法可以访问这些类，我们也称之为 flavor：

* `class pathlib.PurePath(*pathsegments)`

    表示系统路径风格的泛型类（实例化它会创建 `PurePosixPath` 或 `PureWindowsPath` ）：

    ``` python
    >>> PurePath('setup.py')      # 在 Unix 机器上运行
    PurePosixPath('setup.py')
    ```

    `pathsegments` 的每个元素可以是表示路径段的字符串，实现返回字符串的 `os.PathLike` 接口的对象，或者是另一个路径对象：

    ``` python
    >>> PurePath('foo', 'some/path', 'bar')
    PurePosixPath('foo/some/path/bar')
    >>> PurePath(Path('foo'), Path('bar'))
    PurePosixPath('foo/bar')
    ```

    当 `pathsegments` 为空时，假定当前目录：

    ``` python
    >>> PurePath()
    PurePosixPath('.')
    ```

    当给出几个绝对路径时，最后一个被视为一个锚（模仿 `os.path.join()` 的行为）：

    ``` python
    >>> PurePath('/etc', '/usr', 'lib64')
    PurePosixPath('/usr/lib64')
    >>> PureWindowsPath('c:/Windows', 'd:bar')
    PureWindowsPath('d:bar')
    ```

    但是，在 Windows 路径中，更改本地根目录不会丢弃先前的驱动器设置：

    ``` python
    >>> PureWindowsPath('c:/Windows', '/Program Files')
    PureWindowsPath('c:/Program Files')
    ```

    虚假的斜线和单点都会折叠，但双点（'..'）不会折叠，因为这会改变符号链接面前路径的含义：

    ``` python
    >>> PurePath('foo//bar')
    PurePosixPath('foo/bar')
    >>> PurePath('foo/./bar')
    PurePosixPath('foo/bar')
    >>> PurePath('foo/../bar')
    PurePosixPath('foo/../bar')
    ```

    （一种天真的方法会使 `PurePosixPath('foo/../bar')` 等同于 `PurePosixPath('bar')` ，如果 `foo` 是指向另一个目录的符号链接，这是错误的）

    纯路径对象实现了 `os.PathLike` 接口，允许它们在接受接口的任何地方使用。

    > Python 3.6 中已更改：添加了对 `os.PathLike` 接口的支持。

* `class pathlib.PurePosixPath(*pathsegments)`

    `PurePath` 的子类，此路径风格表示非 Windows 文件系统路径：

    ``` python
    >>> PurePosixPath('/etc')
    PurePosixPath('/etc')
    ```

* `class pathlib.PureWindowsPath(*pathsegments)`

    `PurePath` 的子类，此路径风格表示 Windows 文件系统路径：

    ``` python
    >>> PureWindowsPath('c:/Program Files/')
    PureWindowsPath('c:/Program Files')
    ```

无论您运行的是哪个系统，都可以实例化所有这些类，因为它们不提供任何进行系统调用的操作。

## 一般属性

路径是不可变的和可清除的。相同风格的路径是可比较的和可排序的。这些属性尊重风格的案例折叠语义：

``` python
>>> PurePosixPath('foo') == PurePosixPath('FOO')
False
>>> PureWindowsPath('foo') == PureWindowsPath('FOO')
True
>>> PureWindowsPath('FOO') in { PureWindowsPath('foo') }
True
>>> PureWindowsPath('C:') < PureWindowsPath('d:')
True
```

不同风格的路径比较不相等且无法排序：

``` python
>>> PureWindowsPath('foo') == PurePosixPath('foo')
False
>>> PureWindowsPath('foo') < PurePosixPath('foo')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: '<' not supported between instances of 'PureWindowsPath' and 'PurePosixPath'
```

## 操作符

斜杠运算符有助于创建子路径，类似于 `os.path.join()`:

``` python
>>> p = PurePath('/etc')
>>> p
PurePosixPath('/etc')
>>> p / 'init.d' / 'apache2'
PurePosixPath('/etc/init.d/apache2')
>>> q = PurePath('bin')
>>> '/usr' / q
PurePosixPath('/usr/bin')
```

在接受实现 `os.PathLike` 的对象的任何地方都可以使用路径对象：

``` python
>>> import os
>>> p = PurePath('/etc')
>>> os.fspath(p)
'/etc'
```

路径的字符串表示形式是原始文件系统路径本身（以本机形式，例如在 Windows 下使用反斜杠），您可以将其作为字符串传递给任何将文件路径作为函数：

``` python
>>> p = PurePath('/etc')
>>> str(p)
'/etc'
>>> p = PureWindowsPath('c:/Program Files')
>>> str(p)
'c:\\Program Files'
```

类似地，在路径上调用 `bytes` 会将原始文件系统路径作为字节对象，由 `os.fsencode()` 编码：

``` python
>>> bytes(p)
b'/etc'
```

!> 提示： 仅在 Unix 下建议调用字节。在 Windows 下，unicode 表单是文件系统路径的规范表示。

## 访问各个部分

要访问路径的各个“部件”（组件），请使用以下属性：

* `PurePath.parts`

    一个元组，可以访问路径的各个组件：

    ``` python
    >>> p = PurePath('/usr/bin/python3')
    >>> p.parts
    ('/', 'usr', 'bin', 'python3')

    >>> p = PureWindowsPath('c:/Program Files/PSF')
    >>> p.parts
    ('c:\\', 'Program Files', 'PSF')
    ```

    （注意驱动器和本地根重新组合在一个部件中）

## 方法和属性

纯路径提供以下方法和属性：

* `PurePath.drive`

    表示驱动器号或名称的字符串（如果有）：

    ``` python
    >>> PureWindowsPath('c:/Program Files/').drive
    'c:'
    >>> PureWindowsPath('/Program Files/').drive
    ''
    >>> PurePosixPath('/etc').drive
    ''
    ```

    UNC shares 也被视为驱动器：

    ``` python
    >>> PureWindowsPath('//host/share/foo.txt').drive
    '\\\\host\\share'
    ```

* `PurePath.root`

    表示（本地或全局）根的字符串（如果有）：

    ``` python
    >>> PureWindowsPath('c:/Program Files/').root
    '\\'
    >>> PureWindowsPath('c:Program Files/').root
    ''
    >>> PurePosixPath('/etc').root
    '/'
    ```

    UNC shares 总是有根：

    ``` python
    >>> PureWindowsPath('//host/share').root
    '\\'
    ```

* `PurePath.anchor`

    驱动器和根的串联：

    ``` python
    >>> PureWindowsPath('c:/Program Files/').anchor
    'c:\\'
    >>> PureWindowsPath('c:Program Files/').anchor
    'c:'
    >>> PurePosixPath('/etc').anchor
    '/'
    >>> PureWindowsPath('//host/share').anchor
    '\\\\host\\share\\'
    ```

* `PurePath.parents`

    提供对路径逻辑祖先的访问的不可变序列：

    ``` python
    >>> p = PureWindowsPath('c:/foo/bar/setup.py')
    >>> p.parents[0]
    PureWindowsPath('c:/foo/bar')
    >>> p.parents[1]
    PureWindowsPath('c:/foo')
    >>> p.parents[2]
    PureWindowsPath('c:/')
    ```

* `PurePath.parent`

    路径的逻辑父级：

    ``` python
    >>> p = PurePosixPath('/a/b/c/d')
    >>> p.parent
    PurePosixPath('/a/b/c')
    ```

    你无法通过锚点或空路径：

    ``` python
    >>> p = PurePosixPath('/')
    >>> p.parent
    PurePosixPath('/')
    >>> p = PurePosixPath('.')
    >>> p.parent
    PurePosixPath('.')
    ```

    这是一个纯粹的词法操作，因此有以下行为：

    ``` python
    >>> p = PurePosixPath('foo/..')
    >>> p.parent
    PurePosixPath('foo')
    ```

    如果要向上移动任意文件系统路径，建议首先调用 `Path.resolve()` 以解析符号链接并消除“..”组件。

* `PurePath.name`

    表示最终路径组件的字符串，不包括驱动器和根目录（如果有）：

    ``` python
    >>> PurePosixPath('my/library/setup.py').name
    'setup.py'
    ```

    不考虑 UNC 驱动器名称：

    ``` python
    >>> PureWindowsPath('//some/share/setup.py').name
    'setup.py'
    >>> PureWindowsPath('//some/share').name
    ''
    ```

* `PurePath.suffix`

    最终组件的文件扩展名（如果有）：

    ``` python
    >>> PurePosixPath('my/library/setup.py').suffix
    '.py'
    >>> PurePosixPath('my/library.tar.gz').suffix
    '.gz'
    >>> PurePosixPath('my/library').suffix
    ''
    ```

* `PurePath.suffixes`

    路径文件扩展名列表：

    ``` python
    >>> PurePosixPath('my/library.tar.gar').suffixes
    ['.tar', '.gar']
    >>> PurePosixPath('my/library.tar.gz').suffixes
    ['.tar', '.gz']
    >>> PurePosixPath('my/library').suffixes
    []
    ```

* `PurePath.stem`

    最终路径组件，没有后缀：

    ``` python
    >>> PurePosixPath('my/library.tar.gz').stem
    'library.tar'
    >>> PurePosixPath('my/library.tar').stem
    'library'
    >>> PurePosixPath('my/library').stem
    'library'
    ```

* `PurePath.as_posix()`

    返回带正斜杠（/）的路径的字符串表示形式：

    ``` python
    >>> p = PureWindowsPath('c:\\windows')
    >>> str(p)
    'c:\\windows'
    >>> p.as_posix()
    'c:/windows'
    ```

* `PurePath.as_uri()`:

    将路径表示为文件 URI。如果路径不是绝对路径，则引发 `ValueError` 。

    ``` python
    >>> p = PurePosixPath('/etc/passwd')
    >>> p.as_uri()
    'file:///etc/passwd'
    >>> p = PureWindowsPath('c:/Windows')
    >>> p.as_uri()
    'file:///c:/Windows'
    ```

* `PurePath.is_absolute()`

    返回路径是否绝对。如果路径同时具有根和（如果风味允许）驱动器，则该路径被视为绝对路径：

    ``` python
    >>> PurePosixPath('/a/b').is_absolute()
    True
    >>> PurePosixPath('a/b').is_absolute()
    False

    >>> PureWindowsPath('c:/a/b').is_absolute()
    True
    >>> PureWindowsPath('/a/b').is_absolute()
    False
    >>> PureWindowsPath('c:').is_absolute()
    False
    >>> PureWindowsPath('//some/share').is_absolute()
    True
    ```

* `PurePath.is_reserved()`

    使用 `PureWindowsPath` 时，如果路径在 Windows 下被视为保留，则返回 `True` ，否则返回 `False` 。使用 `PurePosixPath` 时，始终返回 `False` 。

    ``` python
    >>> PureWindowsPath('nul').is_reserved()
    True
    >>> PurePosixPath('nul').is_reserved()
    False
    ```

    保留路径上的文件系统调用可能会神秘失败或产生意外影响。

* `PurePath.joinpath(*other)`

    调用此方法相当于将路径依次与每个其他参数组合：

    ``` python
    >>> PurePosixPath('/etc').joinpath('passwd')
    PurePosixPath('/etc/passwd')
    >>> PurePosixPath('/etc').joinpath(PurePosixPath('passwd'))
    PurePosixPath('/etc/passwd')
    >>> PurePosixPath('/etc').joinpath('init.d', 'apache2')
    PurePosixPath('/etc/init.d/apache2')
    >>> PureWindowsPath('c:').joinpath('/Program Files')
    PureWindowsPath('c:/Program Files')
    ```

* `PurePath.match(pattern)`

    将此路径与提供的 glob 风格模式匹配。如果匹配成功则返回 `True` ，否则返回 `False` 。

    如果 `pattern` 是相对的，则路径可以是相对路径或绝对路径，并且匹配是从右侧完成的：

    ``` python
    >>> PurePath('a/b.py').match('*.py')
    True
    >>> PurePath('/a/b/c.py').match('b/*.py')
    True
    >>> PurePath('/a/b/c.py').match('a/*.py')
    False
    ```

    如果 `pattern` 是绝对的，则路径必须是绝对路径，并且整个路径必须匹配：

    ``` python
    >>> PurePath('/a.py').match('/*.py')
    True
    >>> PurePath('a/b.py').match('/*.py')
    False
    ```

    与其他方法一样，可以观察到区分大小写：

    ``` python
    >>> PureWindowsPath('b.py').match('*.PY')
    True
    ```

* `PurePath.relative_to(*other)`

    计算此路径相对于其他路径表示的路径的版本。如果不可能，则会引发 `ValueError` ：

    ``` python
    >>> p = PurePosixPath('/etc/passwd')
    >>> p.relative_to('/')
    PurePosixPath('etc/passwd')
    >>> p.relative_to('/etc')
    PurePosixPath('passwd')
    >>> p.relative_to('/usr')
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "pathlib.py", line 694, in relative_to
        .format(str(self), str(formatted)))
    ValueError: '/etc/passwd' does not start with '/usr'
    ```

* `PurePath.with_name(name)`

    返回名称已更改的新路径。如果原始路径没有名称，则引发 `ValueError` ：

    ``` python
    >>> p = PureWindowsPath('c:/Downloads/pathlib.tar.gz')
    >>> p.with_name('setup.py')
    PureWindowsPath('c:/Downloads/setup.py')
    >>> p = PureWindowsPath('c:/')
    >>> p.with_name('setup.py')
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "/home/antoine/cpython/default/Lib/pathlib.py", line 751, in with_name
        raise ValueError("%r has an empty name" % (self,))
    ValueError: PureWindowsPath('c:/') has an empty name
    ```

* `PurePath.with_suffix(suffix)`

    返回更改后缀的新路径。如果原始路径没有后缀，则会添加新后缀。如果后缀为空字符串，则删除原始后缀：

    ``` python
    >>> p = PureWindowsPath('c:/Downloads/pathlib.tar.gz')
    >>> p.with_suffix('.bz2')
    PureWindowsPath('c:/Downloads/pathlib.tar.bz2')
    >>> p = PureWindowsPath('README')
    >>> p.with_suffix('.txt')
    PureWindowsPath('README.txt')
    >>> p = PureWindowsPath('README.txt')
    >>> p.with_suffix('')
    PureWindowsPath('README')
    ```

# 具体路径(Concrete paths)

具体路径是纯路径类的子类。除了后者提供的操作之外，它们还提供了对路径对象进行系统调用的方法。有三种方法可以实例化具体路径：

``` python
>>> Path('setup.py')
PosixPath('setup.py')
```

``` python
>>> PosixPath('/etc')
PosixPath('/etc')
```

``` python
>>> WindowsPath('c:/Program Files/')
WindowsPath('c:/Program Files')
```

您只能实例化与您的系统对应的类风格（允许对不兼容的路径风格进行系统调用可能导致应用程序中的错误或失败）：

``` python
>>> import os
>>> os.name
'posix'
>>> Path('setup.py')
PosixPath('setup.py')
>>> PosixPath('setup.py')
PosixPath('setup.py')
>>> WindowsPath('setup.py')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "pathlib.py", line 798, in __new__
    % (cls.__name__,))
NotImplementedError: cannot instantiate 'WindowsPath' on your system
```

## 方法

除纯路径方法外，具体路径还提供以下方法。如果系统调用失败，许多这些方法都会引发OSError（例如，因为路径不存在）：

* `classmethod Path.cwd()`

    返回表示当前目录的新路径对象（由os.getcwd（）返回）：

    ``` python
    >>> Path.cwd()
    PosixPath('/home/antoine/pathlib')
    ```

* `classmethod Path.home()`

    返回一个表示用户主目录的新路径对象

    ``` python
    >>> Path.home()
    PosixPath('/home/antoine')
    ```

* `Path.stat()`

    返回有关此路径的信息

    ``` python
    >>> p = Path('setup.py')
    >>> p.stat().st_size
    956
    >>> p.stat().st_mtime
    1327883547.852554
    ```

* `Path.chmod(mode)`

    更改文件模式和权限，例如os.chmod（）：

    ``` python
    >>> p = Path('setup.py')
    >>> p.stat().st_mode
    33277
    >>> p.chmod(0o444)
    >>> p.stat().st_mode
    33060
    ```

* `Path.exists()`

    路径是指向现有文件 or 目录：

    ``` python
    >>> Path('.').exists()
    True
    >>> Path('setup.py').exists()
    True
    >>> Path('/etc').exists()
    True
    >>> Path('nonexistentfile').exists()
    False
    ```

* `Path.expanduser()`

    Return a new path with expanded ~ and ~user constructs, as returned by os.path.expanduser():

    ``` python
    >>> p = PosixPath('~/films/Monty Python')
    >>> p.expanduser()
    PosixPath('/home/eric/films/Monty Python')
    ```

* `Path.glob(pattern)`

    Glob此路径表示的目录中的给定模式，产生所有匹配的文件（任何类型）：

    ``` python
    >>> sorted(Path('.').glob('*.py'))
    [PosixPath('pathlib.py'), PosixPath('setup.py'), PosixPath('test_pathlib.py')]
    >>> sorted(Path('.').glob('*/*.py'))
    [PosixPath('docs/conf.py')]
    ```

    “**”模式表示“此目录和所有子目录，递归”。换句话说，它启用递归通配：

    ``` python
    >>> sorted(Path('.').glob('**/*.py'))
    [PosixPath('build/lib/pathlib.py'),
    PosixPath('docs/conf.py'),
    PosixPath('pathlib.py'),
    PosixPath('setup.py'),
    PosixPath('test_pathlib.py')]
    ```

* `Path.group()`

    返回拥有该文件的组的名称。如果在系统数据库中找不到文件的gid，则引发KeyError。

* `Path.is_dir()`

    如果路径指向目录（或指向目录的符号链接），则返回True，如果指向另一种文件，则返回False。

    如果路径不存在或符号链接损坏，也会返回False;传播其他错误（例如权限错误）。

* `Path.is_file()`

    如果路径指向常规文件（或指向常规文件的符号链接），则返回True;如果指向另一种文件，则返回False。

    如果路径不存在或符号链接损坏，也会返回False;传播其他错误（例如权限错误）。

* `Path.is_mount()`

    如果路径是装入点，则返回True：文件系统中已装入其他文件系统的点。

* `Path.is_symlink()`

    如果路径指向符号链接，则返回True，否则返回False。

* `Path.is_socket()`

    如果路径指向Unix套接字（或指向Unix套接字的符号链接），则返回True，如果指向另一种文件，则返回False。

* `Path.is_fifo()`

    如果路径指向FIFO（或指向FIFO的符号链接），则返回True;如果指向另一种文件，则返回False。

* `Path.is_block_device()`

    如果路径指向块设备（或指向块设备的符号链接），则返回True，如果指向另一种文件，则返回False。

* `Path.is_char_device()`

    如果路径指向字符设备（或指向字符设备的符号链接），则返回True，如果指向另一种文件，则返回False。

* `Path.iterdir()`

    当路径指向目录时，产生目录内容的路径对象：

    ``` python
    >>> p = Path('docs')
    >>> for child in p.iterdir(): child
    ...
    PosixPath('docs/conf.py')
    PosixPath('docs/_templates')
    PosixPath('docs/make.bat')
    PosixPath('docs/index.rst')
    PosixPath('docs/_build')
    PosixPath('docs/_static')
    PosixPath('docs/Makefile')
    ```

* `Path.lchmod(mode)`

    与Path.chmod（）类似，但是，如果路径指向符号链接，则更改符号链接的模式而不是其目标。

* `Path.lstat()`

    与Path.stat（）类似，但是，如果路径指向符号链接，则返回符号链接的信息而不是其目标。

* `Path.mkdir(mode=0o777, parents=False, exist_ok=False)`

    在此给定路径上创建一个新目录。如果给出了mode，则将其与进程'umask值组合以确定文件模式和访问标志。如果路径已存在，则引发FileExistsError。

* `Path.open(mode='r', buffering=-1, encoding=None, errors=None, newline=None)`

    打开路径指向的文件，就像内置的open（）函数一样：

    ``` python
    >>> p = Path('setup.py')
    >>> with p.open() as f:
    ...     f.readline()
    ...
    '#!/usr/bin/env python3\n'
    ```

* `Path.owner()`

    返回拥有该文件的用户的名称。如果在系统数据库中找不到文件的uid，则引发KeyError。

* `Path.read_bytes()`

    将指向文件的二进制内容作为字节对象返回：

    ``` python
    >>> p = Path('my_binary_file')
    >>> p.write_bytes(b'Binary file contents')
    20
    >>> p.read_bytes()
    b'Binary file contents'
    ```

* `Path.read_text(encoding=None, errors=None)`

    将指向文件的已解码内容作为字符串返回：

    ``` python
    >>> p = Path('my_text_file')
    >>> p.write_text('Text file contents')
    18
    >>> p.read_text()
    'Text file contents'
    ```

    该文件已打开然后关闭。可选参数与open（）中的含义相同。

* `Path.rename(target)`

    将此文件或目录重命名为给定目标。在Unix上，如果target存在并且是一个文件，如果用户有权限，它将被静默替换。 target可以是字符串或其他路径对象：

    ``` python
    >>> p = Path('foo')
    >>> p.open('w').write('some text')
    9
    >>> target = Path('bar')
    >>> p.rename(target)
    >>> target.open().read()
    'some text'
    ```

* `Path.replace(target)`

    将此文件或目录重命名为给定目标。如果目标指向现有文件或目录，则将无条件地替换它。

* `Path.resolve(strict=False)`

    使路径绝对，解析任何符号链接。返回一个新的路径对象：

    ``` python
    >>> p = Path()
    >>> p
    PosixPath('.')
    >>> p.resolve()
    PosixPath('/home/antoine/pathlib')
    ```

    “..”组件也被删除（这是唯一的方法）：

    ``` python
    >>> p = Path('docs/../setup.py')
    >>> p.resolve()
    PosixPath('/home/antoine/pathlib/setup.py')
    ```

    如果路径不存在且strict为True，则引发FileNotFoundError。如果strict为False，则尽可能地解析路径，并附加任何余数而不检查它是否存在。如果在解析路径上遇到无限循环，则引发RuntimeError。

* `Path.rglob(pattern)`

    这就像在给定模式前面添加“**”一样调用Path.glob（）：
    
    ``` python
    >>> sorted(Path().rglob("*.py"))
    [PosixPath('build/lib/pathlib.py'),
    PosixPath('docs/conf.py'),
    PosixPath('pathlib.py'),
    PosixPath('setup.py'),
    PosixPath('test_pathlib.py')]
    ```

* `Path.rmdir()`

    删除此目录。该目录必须为空。

* `Path.samefile(other_path)`

    返回此路径是否指向与other_path相同的文件，other_path可以是Path对象，也可以是字符串。语义类似于os.path.samefile（）和os.path.samestat（）。

    如果由于某种原因无法访问任何文件，则可能引发OSError。

    ``` python
    >>> p = Path('spam')
    >>> q = Path('eggs')
    >>> p.samefile(q)
    False
    >>> p.samefile('spam')
    True
    ```

* `Path.symlink_to(target, target_is_directory=False)`

    使此路径成为目标的符号链接。在Windows下，如果链接的目标是目录，则target_is_directory必须为true（默认为False）。在POSIX下，将忽略target_is_directory的值。

    ``` python
    >>> p = Path('mylink')
    >>> p.symlink_to('setup.py')
    >>> p.resolve()
    PosixPath('/home/antoine/pathlib/setup.py')
    >>> p.stat().st_size
    956
    >>> p.lstat().st_size
    8
    ```

* `Path.touch(mode=0o666, exist_ok=True)`

    在此给定路径创建文件。如果给出了mode，则将其与进程'umask值组合以确定文件模式和访问标志。如果文件已存在，则如果exist_ok为true（并且其修改时间更新为当前时间），则函数成功，否则引发FileExistsError。

* `Path.unlink()`

    删除此文件或符号链接。如果路径指向目录，请改用Path.rmdir（）。

* `Path.write_bytes(data)`

    打开以字节模式指向的文件，向其写入数据，然后关闭文件：

    ``` python
    >>> p = Path('my_binary_file')
    >>> p.write_bytes(b'Binary file contents')
    20
    >>> p.read_bytes()
    b'Binary file contents'
    ```

    将覆盖现有的同名文件。

* `Path.write_text(data, encoding=None, errors=None)`

    打开文本模式指向的文件，向其写入数据，然后关闭文件：

    ``` python
    >>> p = Path('my_text_file')
    >>> p.write_text('Text file contents')
    18
    >>> p.read_text()
    'Text file contents'
    ```

# 对应 os 模块中的工具

下面是一个表，它将各种 os 函数映射到它们对应的 `PurePath/Path` 等价物。

?> 尽管 `os.path.relpath()` 和 `PurePath.relative_to()` 具有一些重叠的用例，但它们的语义差异足以保证不会将它们视为等效。

| os 和 os.path | pathlib
|:-----|:---------
| os.path.abspath() | Path.resolve()
| os.chmod() | Path.chmod()
| os.mkdir() | Path.mkdir()
| os.rename() | Path.rename()
| os.replace() | Path.replace()
| os.rmdir() | Path.rmdir()
| os.remove(), os.unlink() | Path.unlink()
| os.getcwd() | Path.cwd()
| os.path.exists() | Path.exists()
| os.path.expanduser() | Path.expanduser() and Path.home()
| os.path.isdir() | Path.is_dir()
| os.path.isfile() | Path.is_file()
| os.path.islink() | Path.is_symlink()
| os.stat() | Path.stat(), Path.owner(), Path.group()
| os.path.isabs() | PurePath.is_absolute()
| os.path.join() | PurePath.joinpath()
| os.path.basename() | PurePath.name
| os.path.dirname() | PurePath.parent
| os.path.samefile() | Path.samefile()
| os.path.splitext() | PurePath.suffix