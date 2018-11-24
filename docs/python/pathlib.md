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

## 方法

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