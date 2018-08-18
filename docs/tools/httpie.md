> [官方原文](https://httpie.org/doc)


# 安装

## macOS

在 macOS 上，HTTPie 可以通过 Homebrew 安装（推荐）：

``` bash
$ brew install httpie
```

## Linux

大多数 Linux 发行版提供了可以使用系统包管理器安装的包，例如：

``` bash
# Debian, Ubuntu, etc.
$ apt-get install httpie

# Fedora
$ dnf install httpie

# CentOS, RHEL, ...
$ yum install httpie

# Arch Linux
$ pacman -S httpie
```

Windows, 等.
-------------

用安装方法（适用于 Windows，Mac OS X，Linux，...，并始终提供最新版本）是使用 pip：

``` bash
# 确保我们拥有最新版本的 pip 和 setuptools：
$ pip install --upgrade pip setuptools

$ pip install --upgrade httpie
```

?> 如果由于某种原因导致 `pip` 安装失败，您可以尝试使用 `easy_install httpie` 。

## version

``` bash
http --version
```

详细信息

``` bash
http --debug
```

# 用法

Hello World:

``` bash
$ http httpie.org
```

概要:

``` bash
$ http [flags] [METHOD] URL [ITEM [ITEM]]
```

另见 `http --help`。

## 例子

自定义 [HTTP 方法](#HTTP-方法)，[HTTP 首部](#HTTP-首部) 和 [JSON](#JSON) 数据:

``` bash
$ http PUT example.org X-API-Token:123 name=John
```

提交[表单](#表单):

``` bash
http -f POST example.org hello=World
```

请参阅使用以下[输出选项](#输出选项)之一发送请求：

``` bash
$ http -v example.org
```

使用 Github API 对[身份验证](#身份验证)问题发表评论：

``` bash
$ http -a USERNAME POST https://api.github.com/repos/jakubroztocil/httpie/issues/83/comments body='HTTPie is awesome! :heart:'
```

使用[重定向输入](#重定向输入)上传文件：

``` bash
$ http example.org < file.json
```

下载文件并通过[重定向输出](#重定向输出)保存：

``` bash
$ http example.org/file > file
```

`wget` 风格下载文件：

``` bash
$ http --download example.org/file
```

使用命名 [session](#session) 在对同一主机的请求之间建立某些方面或通信持久性：

``` bash
$ http --session=logged-in -a username:password httpbin.org/get API-Key:123

$ http --session=logged-in httpbin.org/headers
```

设置自定义 `Host` 首部以解决丢失的 DNS 记录：

``` bash
$ http localhost:8000 Host:example.com
```

# HTTP 方法

HTTP 方法的名称就在 URL 参数之前：

``` bash
$ http DELETE example.org/todos/7
```

其外观类似于发送的实际请求行：

```
DELETE /todos/7 HTTP/1.1
```

当从命令中省略 `METHOD` 参数时，HTTPie 默认为 `GET`（没有请求数据）或 `POST`（带请求数据）。

# 请求 URL

HTTPie 执行请求所需的唯一信息是 URL。`http://` 可以从参数中省略 - `http example.org` 可以工作得很好。

## 查询字符串参数

如果您发现自己在终端上手动构建 URL，您可能会欣赏用于附加 URL 参数的 `param==value` 语法。有了它，您不必担心转义 shell 的 `＆` 分隔符。此外，参数值中的特殊字符也将自动转义（HTTPie 另外要求 URL 已经被转义）。要在 Google 图片上搜索 HTTPie 徽标，您可以使用以下命令：

``` bash
$ http www.google.com search=='HTTPie logo' tbm==isch
GET /?search=HTTPie+logo&tbm=isch HTTP/1.1
```

## localhost 的 URL 快捷方式


此外，支持 localhost 类似 curl 的简写。这意味着，例如 `:3000` 将扩展为 `http://localhost:3000` 如果省略端口，则假定端口 80。

``` bash
$ http :/foo

GET /foo HTTP/1.1
Host: localhost
```

``` bash
$ http :3000/bar

GET /bar HTTP/1.1
Host: localhost:3000
```

``` bash
$ http :

GET / HTTP/1.1
Host: localhost
```

## 自定义默认 scheme


您可以使用 `--default-scheme <URL_SCHEME>` 选项为除 HTTP 之外的其他协议创建快捷方式：

``` bash
$ alias https='http --default-scheme=https'
```


# Request items

有一些不同的 *request item* 类型提供了一种方便的机制来指定 HTTP 首部，简单的 JSON 和表单数据，文件和 URL 参数。

它们是 URL 后指定的键/值对。所有这些都是共同的，它们成为发送的实际请求的一部分，并且它们的类型仅通过使用的分隔符区分：
``:``, ``=``, ``:=``, ``==``, ``@``, ``=@``, 和 ``:=@``. 带有 `@` 的文件路径作为值。

| Item 类型 | 描述 |
|:------------------------
| HTTP 首部 `Name:Value`　| 任意 HTTP 首部，例如 `X-API-Token:123`。
| URL 参数 `name==value` | 将给定的 name/value 对作为查询字符串参数附加到 URL。使用 `==` 分隔符。
| 数据字段 `field=value`, `field=@file.txt` | 请求将数据字段序列化为 JSON 对象（默认），或者进行表单编码（ `--form, -f` ）。
| 原始 JSON 字段 `field:=json`, `field:=@file.json` | 发送 JSON 时有用，一个或多个字段需要是 `Boolean`，`Number`，嵌套 `Object` 或 `Array`，例如，`meals:='["ham","spam"]'` 或者 `pies:=[1,2,3]`（注意引号）。
| 表单文件字段 `field@/dir/file` | 仅适用于 `--form`, `-f`。例如 `screenshot@~/Pictures/img.png`。文件字段的存在导致 `multipart/form-data` 请求。

请注意，数据字段不是指定请求数据的唯一方法：[重定向输入](#重定向输入)是一种用于传递任意数据请求的请求机制。

## 逃避规则

您可以使用 `\` 来转义作为分隔符（或其部分）的字符。例如，`foo\==bar` 将成为数据键/值对（ `foo=` 和 `bar`）而不是 URL 参数。

通常有必要引用这些值，例如： ``foo='bar baz'``.

如果任何字段名称或标题以减号（例如 `-fieldname`）开头，则需要将所有此类 item 放在特殊标记 `--` 之后，以防止与 `--arguments` 混淆：

``` bash
$ http httpbin.org/post  --  -name-starting-with-dash=foo -Unusual-Header:bar

POST /post HTTP/1.1
-Unusual-Header: bar
Content-Type: application/json

{
    "-name-starting-with-dash": "value"
}
```

# JSON

JSON 是现代 Web 服务的通用语言，它也是 HTTPie 默认使用的隐式内容类型。

简单的例子：

``` bash
$ http PUT example.org name=John email=john@example.org

PUT / HTTP/1.1
Accept: application/json, */*
Accept-Encoding: gzip, deflate
Content-Type: application/json
Host: example.org

{
    "name": "John",
    "email": "john@example.org"
}
```

## 默认行为

如果您的命令包含一些数据请求项，则默认情况下将它们序列化为 JSON 对象。HTTPie 还会自动设置以下首部，这两个首部都可以被覆盖：

| - | -
|:--------
| ``Content-Type``  |   ``application/json``
| ``Accept``         | ``application/json, */*``

## 明确的 JSON

无论是否发送数据，都可以使用 `--json，-j` 将 `Accept` 显式设置为 `application/json`（这是通过常用首部符号设置首部的快捷方式：`http url Accept:'application/json, */*'`）。此外，即使 `Content-Type` 是 `text/plain` 或未知，HTTPie 也会尝试检测 JSON 响应。

## 非字符串 JSON 字段

非字符串字段使用 `:=` 分隔符，它允许您将原始 JSON 嵌入到结果对象中。也可以使用 `=@` 和 `:=@` 将文本和原始 JSON 文件嵌入到字段中：

``` bash
$ http PUT api.example.com/person/1 \
    name=John \
    age:=29 married:=false hobbies:='["http", "pies"]' \  # 原始 JSON
    description=@about-john.txt \   # 嵌入文本文件
    bookmarks:=@bookmarks.json      # 嵌入 JSON 文件

PUT /person/1 HTTP/1.1
Accept: application/json, */*
Content-Type: application/json
Host: api.example.com

{
    "age": 29,
    "hobbies": [
        "http",
        "pies"
    ],
    "description": "John is a nice guy who likes pies.",
    "married": false,
    "name": "John",
    "bookmarks": {
        "HTTPie": "http://httpie.org",
    }
}
```

请注意，使用此语法时，命令在发送复杂数据时会变得难以处理。在这种情况下，使用[重定向输入](#重定向输入)总是更好：

``` bash
$ http POST api.example.com/person/1 < person.json
```

# 表单

提交表单与发送 JSON 请求非常相似。通常唯一的区别是添加 ``--form, -f`` 选项, 它确保将数据字段序列化，并将 `Content-Type` 设置为
``application/x-www-form-urlencoded; charset=utf-8``。可以通过[配置](#配置)文件使表单数据成为隐式内容类型而不是 JSON。

## 常规表单

``` bash
$ http --form POST api.example.org/person/1 name='John Smith'

POST /person/1 HTTP/1.1
Content-Type: application/x-www-form-urlencoded; charset=utf-8

name=John+Smith
```

## 文件上传表单

如果存在一个或多个文件字段，则序列化和内容类型为 ``multipart/form-data``:

``` bash
$ http -f POST example.com/jobs name='John Smith' cv@~/Documents/cv.pdf
```

上面的请求与提交以下HTML表单的请求相同：

``` html
<form enctype="multipart/form-data" method="post" action="http://example.com/jobs">
    <input type="text" name="name" />
    <input type="file" name="cv" />
</form>
```

?> 请注意，`@` 用于模拟文件上传表单字段，而 `=@` 只是将文件内容嵌入为常规文本字段值。

# HTTP 首部

要设置自定义首部，您可以使用 `Header:Value` 表示法：

``` bash
$ http example.org  User-Agent:Bacon/1.0  'Cookie:valued-visitor=yes;foo=bar'  \
    X-Foo:Bar  Referer:http://httpie.org/

GET / HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Cookie: valued-visitor=yes;foo=bar
Host: example.org
Referer: http://httpie.org/
User-Agent: Bacon/1.0
X-Foo: Bar
```

## 默认请求首部

HTTPie 设置了几个默认首部：

```
GET / HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
User-Agent: HTTPie/<version>
Host: <taken-from-URL>
```

任何这些 - 除了 `Host` - 都可以被覆盖，其中一些未设置。

## 空标题和标题取消设置

要取消设置先前指定的首部（例如默认首部之一），请使用 `Header:`:

``` bash
$ http httpbin.org/headers Accept: User-Agent:
```

要发送空值的首部，请使用 `Header;`：

``` bash
$ http httpbin.org/headers 'Header;'
```

# Cookies

HTTP clients send cookies to the server as regular `HTTP headers`_. That means,
HTTPie does not offer any special syntax for specifying cookies — the usual
``Header:Value`` notation is used:


Send a single cookie:

.. code-block:: bash

    $ http example.org Cookie:sessionid=foo

.. code-block:: http

    GET / HTTP/1.1
    Accept: */*
    Accept-Encoding: gzip, deflate
    Connection: keep-alive
    Cookie: sessionid=foo
    Host: example.org
    User-Agent: HTTPie/0.9.9


Send multiple cookies
(note the header is quoted to prevent the shell from interpreting the ``;``):

.. code-block:: bash

    $ http example.org 'Cookie:sessionid=foo;another-cookie=bar'

.. code-block:: http

    GET / HTTP/1.1
    Accept: */*
    Accept-Encoding: gzip, deflate
    Connection: keep-alive
    Cookie: sessionid=foo;another-cookie=bar
    Host: example.org
    User-Agent: HTTPie/0.9.9


If you often deal with cookies in your requests, then chances are you'd appreciate
the `sessions`_ feature.


Authentication
==============

The currently supported authentication schemes are Basic and Digest
(see `auth plugins`_ for more). There are two flags that control authentication:

===================     ======================================================
``--auth, -a``          Pass a ``username:password`` pair as
                        the argument. Or, if you only specify a username
                        (``-a username``), you'll be prompted for
                        the password before the request is sent.
                        To send an empty password, pass ``username:``.
                        The ``username:password@hostname`` URL syntax is
                        supported as well (but credentials passed via ``-a``
                        have higher priority).

``--auth-type, -A``     Specify the auth mechanism. Possible values are
                        ``basic`` and ``digest``. The default value is
                        ``basic`` so it can often be omitted.
===================     ======================================================



Basic auth
----------


.. code-block:: bash

    $ http -a username:password example.org


Digest auth
-----------


.. code-block:: bash

    $ http -A digest -a username:password example.org


Password prompt
---------------

.. code-block:: bash

    $ http -a username example.org


``.netrc``
----------

Authentication information from your ``~/.netrc`` file is honored as well:

.. code-block:: bash

    $ cat ~/.netrc
    machine httpbin.org
    login httpie
    password test

    $ http httpbin.org/basic-auth/httpie/test
    HTTP/1.1 200 OK
    [...]


Auth plugins
------------

Additional authentication mechanism can be installed as plugins.
They can be found on the `Python Package Index <https://pypi.python.org/pypi?%3Aaction=search&term=httpie&submit=search>`_.
Here's a few picks:

* `httpie-api-auth <https://github.com/pd/httpie-api-auth>`_: ApiAuth
* `httpie-aws-auth <https://github.com/httpie/httpie-aws-auth>`_: AWS / Amazon S3
* `httpie-edgegrid <https://github.com/akamai-open/httpie-edgegrid>`_: EdgeGrid
* `httpie-hmac-auth <https://github.com/guardian/httpie-hmac-auth>`_: HMAC
* `httpie-jwt-auth <https://github.com/teracyhq/httpie-jwt-auth>`_: JWTAuth (JSON Web Tokens)
* `httpie-negotiate <https://github.com/ndzou/httpie-negotiate>`_: SPNEGO (GSS Negotiate)
* `httpie-ntlm <https://github.com/httpie/httpie-ntlm>`_: NTLM (NT LAN Manager)
* `httpie-oauth <https://github.com/httpie/httpie-oauth>`_: OAuth
* `requests-hawk <https://github.com/mozilla-services/requests-hawk>`_: Hawk




HTTP redirects
==============

By default, HTTP redirects are not followed and only the first
response is shown:


.. code-block:: bash

    $ http httpbin.org/redirect/3


Follow ``Location``
-------------------

To instruct HTTPie to follow the ``Location`` header of ``30x`` responses
and show the final response instead, use the ``--follow, -F`` option:


.. code-block:: bash

    $ http --follow httpbin.org/redirect/3


Showing intermediary redirect responses
---------------------------------------

If you additionally wish to see the intermediary requests/responses,
then use the ``--all`` option as well:


.. code-block:: bash

    $ http --follow --all httpbin.org/redirect/3



Limiting maximum redirects followed
-----------------------------------

To change the default limit of maximum ``30`` redirects, use the
``--max-redirects=<limit>`` option:


.. code-block:: bash

    $ http --follow --all --max-redirects=5 httpbin.org/redirect/3


Proxies
=======

You can specify proxies to be used through the ``--proxy`` argument for each
protocol (which is included in the value in case of redirects across protocols):

.. code-block:: bash

    $ http --proxy=http:http://10.10.1.10:3128 --proxy=https:https://10.10.1.10:1080 example.org


With Basic authentication:

.. code-block:: bash

    $ http --proxy=http:http://user:pass@10.10.1.10:3128 example.org


Environment variables
---------------------

You can also configure proxies by environment variables ``HTTP_PROXY`` and
``HTTPS_PROXY``, and the underlying Requests library will pick them up as well.
If you want to disable proxies configured through the environment variables for
certain hosts, you can specify them in ``NO_PROXY``.

In your ``~/.bash_profile``:

.. code-block:: bash

 export HTTP_PROXY=http://10.10.1.10:3128
 export HTTPS_PROXY=https://10.10.1.10:1080
 export NO_PROXY=localhost,example.com


SOCKS
-----

Homebrew-installed HTTPie comes with SOCKS proxy support out of the box. To enable SOCKS proxy support for non-Homebrew  installations, you'll need to install ``requests[socks]`` manually using ``pip``:


.. code-block:: bash

    $ pip install -U requests[socks]

Usage is the same as for other types of `proxies`_:

.. code-block:: bash

    $ http --proxy=http:socks5://user:pass@host:port --proxy=https:socks5://user:pass@host:port example.org


HTTPS
=====


Server SSL certificate verification
-----------------------------------

To skip the host's SSL certificate verification, you can pass ``--verify=no``
(default is ``yes``):

.. code-block:: bash

    $ http --verify=no https://example.org


Custom CA bundle
----------------

You can also use ``--verify=<CA_BUNDLE_PATH>`` to set a custom CA bundle path:

.. code-block:: bash

    $ http --verify=/ssl/custom_ca_bundle https://example.org



Client side SSL certificate
---------------------------
To use a client side certificate for the SSL communication, you can pass
the path of the cert file with ``--cert``:

.. code-block:: bash

    $ http --cert=client.pem https://example.org


If the private key is not contained in the cert file you may pass the
path of the key file with ``--cert-key``:

.. code-block:: bash

    $ http --cert=client.crt --cert-key=client.key https://example.org


SSL version
-----------

Use the ``--ssl=<PROTOCOL>`` to specify the desired protocol version to use.
This will default to SSL v2.3 which will negotiate the highest protocol that both
the server and your installation of OpenSSL support. The available protocols
are ``ssl2.3``, ``ssl3``, ``tls1``, ``tls1.1``, ``tls1.2``. (The actually
available set of protocols may vary depending on your OpenSSL installation.)

.. code-block:: bash

    # Specify the vulnerable SSL v3 protocol to talk to an outdated server:
    $ http --ssl=ssl3 https://vulnerable.example.org


SNI (Server Name Indication)
----------------------------

If you use HTTPie with `Python version`_ lower than 2.7.9
(can be verified with ``http --debug``) and need to talk to servers that
use SNI (Server Name Indication) you need to install some additional
dependencies:

.. code-block:: bash

    $ pip install --upgrade requests[security]


You can use the following command to test SNI support:

.. code-block:: bash

    $ http https://sni.velox.ch


Output options
==============

By default, HTTPie only outputs the final response and the whole response
message is printed (headers as well as the body). You can control what should
be printed via several options:

=================   =====================================================
``--headers, -h``   Only the response headers are printed.
``--body, -b``      Only the response body is printed.
``--verbose, -v``   Print the whole HTTP exchange (request and response).
                    This option also enables ``--all`` (see below).
``--print, -p``     Selects parts of the HTTP exchange.
=================   =====================================================

``--verbose`` can often be useful for debugging the request and generating
documentation examples:

.. code-block:: bash

    $ http --verbose PUT httpbin.org/put hello=world
    PUT /put HTTP/1.1
    Accept: application/json, */*
    Accept-Encoding: gzip, deflate
    Content-Type: application/json
    Host: httpbin.org
    User-Agent: HTTPie/0.2.7dev

    {
        "hello": "world"
    }


    HTTP/1.1 200 OK
    Connection: keep-alive
    Content-Length: 477
    Content-Type: application/json
    Date: Sun, 05 Aug 2012 00:25:23 GMT
    Server: gunicorn/0.13.4

    {
        […]
    }


What parts of the HTTP exchange should be printed
-------------------------------------------------

All the other `output options`_ are under the hood just shortcuts for
the more powerful ``--print, -p``. It accepts a string of characters each
of which represents a specific part of the HTTP exchange:

==========  ==================
Character   Stands for
==========  ==================
``H``       request headers
``B``       request body
``h``       response headers
``b``       response body
==========  ==================

Print request and response headers:

.. code-block:: bash

    $ http --print=Hh PUT httpbin.org/put hello=world


Viewing intermediary requests/responses
---------------------------------------

To see all the HTTP communication, i.e. the final request/response as
well as any possible  intermediary requests/responses, use the ``--all``
option. The intermediary HTTP communication include followed redirects
(with ``--follow``), the first unauthorized request when HTTP digest
authentication is used (``--auth=digest``), etc.

.. code-block:: bash

    # Include all responses that lead to the final one:
    $ http --all --follow httpbin.org/redirect/3


The intermediary requests/response are by default formatted according to
``--print, -p`` (and its shortcuts described above). If you'd like to change
that, use the ``--history-print, -P`` option. It takes the same
arguments as ``--print, -p`` but applies to the intermediary requests only.


.. code-block:: bash

    # Print the intermediary requests/responses differently than the final one:
    $ http -A digest -a foo:bar --all -p Hh -P H httpbin.org/digest-auth/auth/foo/bar


Conditional body download
-------------------------

As an optimization, the response body is downloaded from the server
only if it's part of the output. This is similar to performing a ``HEAD``
request, except that it applies to any HTTP method you use.

Let's say that there is an API that returns the whole resource when it is
updated, but you are only interested in the response headers to see the
status code after an update:

.. code-block:: bash

    $ http --headers PATCH example.org/Really-Huge-Resource name='New Name'


Since we are only printing the HTTP headers here, the connection to the server
is closed as soon as all the response headers have been received.
Therefore, bandwidth and time isn't wasted downloading the body
which you don't care about. The response headers are downloaded always,
even if they are not part of the output


Redirected Input
================

The universal method for passing request data is through redirected ``stdin``
(standard input)—piping. Such data is buffered and then with no further
processing used as the request body. There are multiple useful ways to use
piping:

Redirect from a file:

.. code-block:: bash

    $ http PUT example.com/person/1 X-API-Token:123 < person.json


Or the output of another program:

.. code-block:: bash

    $ grep '401 Unauthorized' /var/log/httpd/error_log | http POST example.org/intruders


You can use ``echo`` for simple data:

.. code-block:: bash

    $ echo '{"name": "John"}' | http PATCH example.com/person/1 X-API-Token:123


You can even pipe web services together using HTTPie:

.. code-block:: bash

    $ http GET https://api.github.com/repos/jakubroztocil/httpie | http POST httpbin.org/post


You can use ``cat`` to enter multiline data on the terminal:

.. code-block:: bash

    $ cat | http POST example.com
    <paste>
    ^D


.. code-block:: bash

    $ cat | http POST example.com/todos Content-Type:text/plain
    - buy milk
    - call parents
    ^D


On OS X, you can send the contents of the clipboard with ``pbpaste``:

.. code-block:: bash

    $ pbpaste | http PUT example.com


Passing data through ``stdin`` cannot be combined with data fields specified
on the command line:


.. code-block:: bash

    $ echo 'data' | http POST example.org more=data   # This is invalid


To prevent HTTPie from reading ``stdin`` data you can use the
``--ignore-stdin`` option.


Request data from a filename
----------------------------

An alternative to redirected ``stdin`` is specifying a filename (as
``@/path/to/file``) whose content is used as if it came from ``stdin``.

It has the advantage that the ``Content-Type``
header is automatically set to the appropriate value based on the
filename extension. For example, the following request sends the
verbatim contents of that XML file with ``Content-Type: application/xml``:

.. code-block:: bash

    $ http PUT httpbin.org/put @/data/file.xml


Terminal output
===============

HTTPie does several things by default in order to make its terminal output
easy to read.


Colors and formatting
---------------------

Syntax highlighting is applied to HTTP headers and bodies (where it makes
sense). You can choose your preferred color scheme via the ``--style`` option
if you don't like the default one (see ``$ http --help`` for the possible
values).

Also, the following formatting is applied:

* HTTP headers are sorted by name.
* JSON data is indented, sorted by keys, and unicode escapes are converted
  to the characters they represent.

One of these options can be used to control output processing:

====================   ========================================================
``--pretty=all``       Apply both colors and formatting.
                       Default for terminal output.
``--pretty=colors``    Apply colors.
``--pretty=format``    Apply formatting.
``--pretty=none``      Disables output processing.
                       Default for redirected output.
====================   ========================================================

Binary data
-----------

Binary data is suppressed for terminal output, which makes it safe to perform
requests to URLs that send back binary data. Binary data is suppressed also in
redirected, but prettified output. The connection is closed as soon as we know
that the response body is binary,

.. code-block:: bash

    $ http example.org/Movie.mov


You will nearly instantly see something like this:

.. code-block:: http

    HTTP/1.1 200 OK
    Accept-Ranges: bytes
    Content-Encoding: gzip
    Content-Type: video/quicktime
    Transfer-Encoding: chunked

    +-----------------------------------------+
    | NOTE: binary data not shown in terminal |
    +-----------------------------------------+


Redirected output
=================

HTTPie uses a different set of defaults for redirected output than for
`terminal output`_. The differences being:

* Formatting and colors aren't applied (unless ``--pretty`` is specified).
* Only the response body is printed (unless one of the `output options`_ is set).
* Also, binary data isn't suppressed.

The reason is to make piping HTTPie's output to another programs and
downloading files work with no extra flags. Most of the time, only the raw
response body is of an interest when the output is redirected.

Download a file:

.. code-block:: bash

    $ http example.org/Movie.mov > Movie.mov


Download an image of Octocat, resize it using ImageMagick, upload it elsewhere:

.. code-block:: bash

    $ http octodex.github.com/images/original.jpg | convert - -resize 25% -  | http example.org/Octocats


Force colorizing and formatting, and show both the request and the response in
``less`` pager:

.. code-block:: bash

    $ http --pretty=all --verbose example.org | less -R


The ``-R`` flag tells ``less`` to interpret color escape sequences included
HTTPie`s output.

You can create a shortcut for invoking HTTPie with colorized and paged output
by adding the following to your ``~/.bash_profile``:

.. code-block:: bash

    function httpless {
        # `httpless example.org'
        http --pretty=all --print=hb "$@" | less -R;
    }


Download mode
=============

HTTPie features a download mode in which it acts similarly to ``wget``.

When enabled using the ``--download, -d`` flag, response headers are printed to
the terminal (``stderr``), and a progress bar is shown while the response body
is being saved to a file.

.. code-block:: bash

    $ http --download https://github.com/jakubroztocil/httpie/archive/master.tar.gz

.. code-block:: http

    HTTP/1.1 200 OK
    Content-Disposition: attachment; filename=httpie-master.tar.gz
    Content-Length: 257336
    Content-Type: application/x-gzip

    Downloading 251.30 kB to "httpie-master.tar.gz"
    Done. 251.30 kB in 2.73862s (91.76 kB/s)


Downloaded file name
--------------------

If not provided via ``--output, -o``, the output filename will be determined
from ``Content-Disposition`` (if available), or from the URL and
``Content-Type``. If the guessed filename already exists, HTTPie adds a unique
suffix to it.


Piping while downloading
------------------------

You can also redirect the response body to another program while the response
headers and progress are still shown in the terminal:

.. code-block:: bash

    $ http -d https://github.com/jakubroztocil/httpie/archive/master.tar.gz |  tar zxf -



Resuming downloads
------------------

If ``--output, -o`` is specified, you can resume a partial download using the
``--continue, -c`` option. This only works with servers that support
``Range`` requests and ``206 Partial Content`` responses. If the server doesn't
support that, the whole file will simply be downloaded:

.. code-block:: bash

    $ http -dco file.zip example.org/file

Other notes
-----------

* The ``--download`` option only changes how the response body is treated.
* You can still set custom headers, use sessions, ``--verbose, -v``, etc.
* ``--download`` always implies ``--follow`` (redirects are followed).
* HTTPie exits with status code ``1`` (error) if the body hasn't been fully
  downloaded.
* ``Accept-Encoding`` cannot be set with ``--download``.


Streamed responses
==================

Responses are downloaded and printed in chunks which allows for streaming
and large file downloads without using too much memory. However, when
`colors and formatting`_ is applied, the whole response is buffered and only
then processed at once.


Disabling buffering
-------------------

You can use the ``--stream, -S`` flag to make two things happen:

1. The output is flushed in much smaller chunks without any buffering,
   which makes HTTPie behave kind of like ``tail -f`` for URLs.

2. Streaming becomes enabled even when the output is prettified: It will be
   applied to each line of the response and flushed immediately. This makes
   it possible to have a nice output for long-lived requests, such as one
   to the Twitter streaming API.


Examples use cases
------------------

Prettified streamed response:

.. code-block:: bash

    $ http --stream -f -a YOUR-TWITTER-NAME https://stream.twitter.com/1/statuses/filter.json track='Justin Bieber'


Streamed output by small chunks alá ``tail -f``:

.. code-block:: bash

    # Send each new tweet (JSON object) mentioning "Apple" to another
    # server as soon as it arrives from the Twitter streaming API:
    $ http --stream -f -a YOUR-TWITTER-NAME https://stream.twitter.com/1/statuses/filter.json track=Apple \
    | while read tweet; do echo "$tweet" | http POST example.org/tweets ; done

Sessions
========

By default, every request HTTPie makes is completely independent of any
previous ones to the same host.


However, HTTPie also supports persistent
sessions via the ``--session=SESSION_NAME_OR_PATH`` option. In a session,
custom `HTTP headers`_ (except for the ones starting with ``Content-`` or ``If-``),
`authentication`_, and `cookies`_
(manually specified or sent by the server) persist between requests
to the same host.


.. code-block:: bash

    # Create a new session
    $ http --session=/tmp/session.json example.org API-Token:123

    # Re-use an existing session — API-Token will be set:
    $ http --session=/tmp/session.json example.org


All session data, including credentials, cookie data,
and custom headers are stored in plain text.
That means session files can also be created and edited manually in a text
editor—they are regular JSON. It also means that they can be read by anyone
who has access to the session file.


Named sessions
--------------


You can create one or more named session per host. For example, this is how
you can create a new session named ``user1`` for ``example.org``:

.. code-block:: bash

    $ http --session=user1 -a user1:password example.org X-Foo:Bar

From now on, you can refer to the session by its name. When you choose to
use the session again, any previously specified authentication or HTTP headers
will automatically be set:

.. code-block:: bash

    $ http --session=user1 example.org

To create or reuse a different session, simple specify a different name:

.. code-block:: bash

    $ http --session=user2 -a user2:password example.org X-Bar:Foo

Named sessions' data is stored in JSON files in the directory
``~/.httpie/sessions/<host>/<name>.json``
(``%APPDATA%\httpie\sessions\<host>\<name>.json`` on Windows).


Anonymous sessions
------------------

Instead of a name, you can also directly specify a path to a session file. This
allows for sessions to be re-used across multiple hosts:

.. code-block:: bash

    $ http --session=/tmp/session.json example.org
    $ http --session=/tmp/session.json admin.example.org
    $ http --session=~/.httpie/sessions/another.example.org/test.json example.org
    $ http --session-read-only=/tmp/session.json example.org


Readonly session
----------------

To use an existing session file without updating it from the request/response
exchange once it is created, specify the session name via
``--session-read-only=SESSION_NAME_OR_PATH`` instead.


Config
======

HTTPie uses a simple JSON config file.



Config file location
--------------------


The default location of the configuration file is ``~/.httpie/config.json``
(or ``%APPDATA%\httpie\config.json`` on Windows). The config directory
location can be changed by setting the ``HTTPIE_CONFIG_DIR``
environment variable. To view the exact location run ``http --debug``.

Configurable options
--------------------

The JSON file contains an object with the following keys:


``default_options``
~~~~~~~~~~~~~~~~~~~


An ``Array`` (by default empty) of default options that should be applied to
every invocation of HTTPie.

For instance, you can use this option to change the default style and output
options: ``"default_options": ["--style=fruity", "--body"]`` Another useful
default option could be ``"--session=default"`` to make HTTPie always
use `sessions`_ (one named ``default`` will automatically be used).
Or you could change the implicit request content type from JSON to form by
adding ``--form`` to the list.


``__meta__``
~~~~~~~~~~~~

HTTPie automatically stores some of its metadata here. Please do not change.



Un-setting previously specified options
---------------------------------------

Default options from the config file, or specified any other way,
can be unset for a particular invocation via ``--no-OPTION`` arguments passed
on the command line (e.g., ``--no-style`` or ``--no-session``).



Scripting
=========

When using HTTPie from shell scripts, it can be handy to set the
``--check-status`` flag. It instructs HTTPie to exit with an error if the
HTTP status is one of ``3xx``, ``4xx``, or ``5xx``. The exit status will
be ``3`` (unless ``--follow`` is set), ``4``, or ``5``,
respectively.

.. code-block:: bash

    #!/bin/bash

    if http --check-status --ignore-stdin --timeout=2.5 HEAD example.org/health &> /dev/null; then
        echo 'OK!'
    else
        case $? in
            2) echo 'Request timed out!' ;;
            3) echo 'Unexpected HTTP 3xx Redirection!' ;;
            4) echo 'HTTP 4xx Client Error!' ;;
            5) echo 'HTTP 5xx Server Error!' ;;
            6) echo 'Exceeded --max-redirects=<n> redirects!' ;;
            *) echo 'Other Error!' ;;
        esac
    fi


Best practices
--------------

The default behaviour of automatically reading ``stdin`` is typically not
desirable during non-interactive invocations. You most likely want
use the ``--ignore-stdin`` option to disable it.

It is a common gotcha that without this option HTTPie seemingly hangs.
What happens is that when HTTPie is invoked for example from a cron job,
``stdin`` is not connected to a terminal.
Therefore, rules for `redirected input`_ apply, i.e., HTTPie starts to read it
expecting that the request body will be passed through.
And since there's no data nor ``EOF``, it will be stuck. So unless you're
piping some data to HTTPie, this flag should be used in scripts.

Also, it might be good to override the default ``30`` second ``--timeout`` to
something that suits you.



Meta
====

Interface design
----------------

The syntax of the command arguments closely corresponds to the actual HTTP
requests sent over the wire. It has the advantage  that it's easy to remember
and read. It is often possible to translate an HTTP request to an HTTPie
argument list just by inlining the request elements. For example, compare this
HTTP request:

.. code-block:: http

    POST /collection HTTP/1.1
    X-API-Key: 123
    User-Agent: Bacon/1.0
    Content-Type: application/x-www-form-urlencoded

    name=value&name2=value2


with the HTTPie command that sends it:

.. code-block:: bash

    $ http -f POST example.org/collection \
      X-API-Key:123 \
      User-Agent:Bacon/1.0 \
      name=value \
      name2=value2


Notice that both the order of elements and the syntax is very similar,
and that only a small portion of the command is used to control HTTPie and
doesn't directly correspond to any part of the request (here it's only ``-f``
asking HTTPie to send a form request).

The two modes, ``--pretty=all`` (default for terminal) and ``--pretty=none``
(default for redirected output), allow for both user-friendly interactive use
and usage from scripts, where HTTPie serves as a generic HTTP client.

As HTTPie is still under heavy development, the existing command line
syntax and some of the ``--OPTIONS`` may change slightly before
HTTPie reaches its final version ``1.0``. All changes are recorded in the
`change log`_.