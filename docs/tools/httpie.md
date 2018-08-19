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

使用命名 [session](#Session) 在对同一主机的请求之间建立某些方面或通信持久性：

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

HTTP 客户端将 cookie 作为常规 HTTP 首部发送到服务器。这意味着，HTTPie 不提供任何指定 cookie 的特殊语法 - 通常使用 `Header:Value` ：

发送一个 cookie：

``` bash
 $ http example.org Cookie:sessionid=foo

GET / HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Connection: keep-alive
Cookie: sessionid=foo
Host: example.org
User-Agent: HTTPie/0.9.9
```

发送多个 cookie（注意用引号引起来，防止 shell 转义）：

``` bash
$ http example.org 'Cookie:sessionid=foo;another-cookie=bar'

GET / HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Connection: keep-alive
Cookie: sessionid=foo;another-cookie=bar
Host: example.org
User-Agent: HTTPie/0.9.9
```

如果您经常在请求中处理 cookie，那么您很可能会喜欢 [session](#Session) 功能。


# 身份验证

当前支持的身份验证方案是 Basic 和 Digest 。有两个控制身份验证的标志：

| - | - |
|:-----------
| `--auth, -a` | 传递 `username:password` 对作为参数。或者，如果您只指定<br>用户名（`-a username`），则会在发送请求之前提示您<br>输入密码。要发送空密码，请传递 `username:`。还支持<br> `username:password@hostname` URL 语法。<br>（但通过 `-a` 传递的凭证具有更高的优先级）
| `--auth-type, -A` | 指定身份验证机制。可能的值是 `basic` 和 `digest`。<br>默认值为 `basic`，因此通常可以省略。

## Basic 认证

``` bash
$ http -a username:password example.org
```

## Digest 认证

``` bash
$ http -A digest -a username:password example.org
```


## 单独输入密码

``` bash
$ http -a username example.org
```

## .netrc


也支持来自 `~/.netrc` 文件的授权信息：

``` bash
$ cat ~/.netrc
machine httpbin.org
login httpie
password test

$ http httpbin.org/basic-auth/httpie/test
HTTP/1.1 200 OK
[...]
```

## Auth 插件

可以将其他身份验证机制安装为插件。它们可以在 [Python Package Index](https://pypi.python.org/pypi?%3Aaction=search&term=httpie&submit=search) 中找到。这里有几个选择：

* [httpie-api-auth](https://github.com/pd/httpie-api-auth>): ApiAuth
* [httpie-aws-auth](https://github.com/httpie/httpie-aws-auth): AWS / Amazon S3
* [httpie-edgegrid](https://github.com/akamai-open/httpie-edgegrid): EdgeGrid
* [httpie-hmac-auth](https://github.com/guardian/httpie-hmac-auth): HMAC
* [httpie-jwt-auth](https://github.com/teracyhq/httpie-jwt-auth): JWTAuth (JSON Web Tokens)
* [httpie-negotiate](https://github.com/ndzou/httpie-negotiate): SPNEGO (GSS Negotiate)
* [httpie-ntlm](https://github.com/httpie/httpie-ntlm): NTLM (NT LAN Manager)
* [httpie-oauth](https://github.com/httpie/httpie-oauth): OAuth
* [requests-hawk](https://github.com/mozilla-services/requests-hawk): Hawk

# HTTP 重定向

默认情况下，不会遵循 HTTP 重定向，只显示第一个响应：

``` bash
$ http httpbin.org/redirect/3
```

## 跟随 Location

要指示 HTTPie 遵循 `30x` 响应的 `Location` 首部并显示最终响应，请使用 `--follow, -F` 选项：

``` bash
$ http --follow httpbin.org/redirect/3
```

## 显示中间重定向响应

如果您还希望查看中间的 requests/responses，那么再加上 `--all` 选项：

``` bash
$ http --follow --all httpbin.org/redirect/3
```

## 限制最大重定向次数

要更改最多 `30` 个重定向的默认限制，请使用 `--max-redirects=<limit>` 选项：

``` bash
$ http --follow --all --max-redirects=5 httpbin.org/redirect/3
```

# 代理

您可以通过每个协议的 `--proxy` 参数指定要使用的代理（在跨协议重定向的情况下包含在值中）：

``` bash
$ http --proxy=http:http://10.10.1.10:3128 --proxy=https:https://10.10.1.10:1080 example.org
```

使用 Basic 认证：

``` bash
$ http --proxy=http:http://user:pass@10.10.1.10:3128 example.org
```

## 环境变量

您还可以通过环境变量 `HTTP_PROXY` 和 `HTTPS_PROXY` 配置代理，并且底层请求库也将选择它们。如果要禁用某些主机地址的代理，可以在 `NO_PROXY` 中指定它们。

在你的 `~/.bash_profile` 中：

``` bash
export HTTP_PROXY=http://10.10.1.10:3128
export HTTPS_PROXY=https://10.10.1.10:1080
export NO_PROXY=localhost,example.com
```

## SOCKS

Homebrew 安装的 HTTPie 开箱即用 SOCKS 代理支持。要启用 SOCKS 代理支持，请使用 `pip` 安装 `requests[socks]`：

``` bash
$ pip install -U requests[socks]
```

用法与其他类型的代理相同：

``` bash
$ http --proxy=http:socks5://user:pass@host:port --proxy=https:socks5://user:pass@host:port example.org
```

# HTTPS

## 服务器 SSL 证书验证

要跳过主机的 SSL 证书验证，您可以传递 `--verify=no`（默认为 `yes`）：

``` bash
$ http --verify=no https://example.org
```

## 自定义 CA bundle


您还可以使用 `--verify=<CA_BUNDLE_PATH>` 来设置自定义 CA bundle 路径：

``` bash
$ http --verify=/ssl/custom_ca_bundle https://example.org
```

## 客户端 SSL 证书

要使用客户端证书进行 SSL 通信，可以使用 `--cert` 传递 cert 文件的路径：

``` bash
$ http --cert=client.pem https://example.org
```

如果私钥未包含在 cert 文件中，您可以使用 `--cert-key` 传递密钥文件的路径：

``` bash
$ http --cert=client.crt --cert-key=client.key https://example.org
```

## SSL 版本

使用 `--ssl=<PROTOCOL>` 指定要使用的所需协议版本。默认为 SSL v2.3，它是协商服务器和你安装的 OpenSSL 支持的最高协议。可用的协议有 ``ssl2.3``, ``ssl3``, ``tls1``, ``tls1.1``, ``tls1.2``。（实际可用的协议集因你安装的 OpenSSL 而异。）

``` bash
# Specify the vulnerable SSL v3 protocol to talk to an outdated server:
$ http --ssl=ssl3 https://vulnerable.example.org
```

## SNI (Server Name Indication)

如果您使用的 HTTPie 中的 Python 版本低于 2.7.9（可以使用 `http --debug` 验证）并且需要与使用 SNI 的服务器通信，则需要安装一些其他依赖项：

``` bash
$ pip install --upgrade requests[security]
```

您可以使用以下命令测试 SNI 支持：

``` bash
$ http https://sni.velox.ch
```

# 输出选项

认情况下，HTTPie 仅输出最终响应，并打印整个响应消息（ headers 以及 body）。您可以通过以下几个选项控制应该打印的内容：

| - | - |
|:---------
| `--headers, -h` | 仅打印响应首部。|
| `--body, -b` | 仅打印响应正文。|
| `--verbose, -v` | 打印整个 HTTP 交换（请求和响应）。<br>此选项还启用 `--all`（见下文）。|
| `--print, -p` | 选择 HTTP 交换的部分。|

``--verbose`` 通常可用于调试请求和生成文档示例：

``` bash
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
```

## 应该打印 HTTP 交换的哪些部分

所有其他输出选项都只是 `--print, -p` 的快捷方式。它接受一个字符串，每个字符代表 HTTP 交换的特定部分：

| 字符  | 代表 |
|:--------------------------
|``H``   |    request headers
|``B``     |  request body
|``h``    |   response headers
|``b``   |    response body

打印请求和响应首部：

``` bash
$ http --print=Hh PUT httpbin.org/put hello=world
```

## 查看中间请求/响应

要查看所有 HTTP 通信，即最终 请求/响应 以及任何可能的中间 请求/响应，请使用 `--all` 选项。中间 HTTP 通信包括重定向（使用 `--follow`），使用 HTTP digest 认证时的第一个未授权请求（ `--auth=digest` ）等。

``` bash
# Include all responses that lead to the final one:
$ http --all --follow httpbin.org/redirect/3
```

中间 请求/响应 默认根据 `--print, -p` 格式化（及其上述快捷方式）。如果您想更改它，请使用 `--history-print, -P` 选项。它采用与 `--print, -p` 相同的参数，但仅适用于中间请求。

``` bash
# Print the intermediary requests/responses differently than the final one:
$ http -A digest -a foo:bar --all -p Hh -P H httpbin.org/digest-auth/auth/foo/bar
```

## 有条件的 body 下载

作为优化，仅当响应主体是输出的一部分时才从服务器下载响应主体。这与执行 `HEAD` 请求类似，不同之处在于它适用于您使用的任何 HTTP 方法。

假设有一个 API 在更新时返回整个资源，但您只对响应首部感兴趣，以便在更新后查看状态代码：

``` bash
$ http --headers PATCH example.org/Really-Huge-Resource name='New Name'
```

由于我们仅在此处打印 HTTP 首部，因此只要收到所有响应首部，就会关闭与服务器的连接。因此，带宽和时间不会浪费下载您不关心的主体。 始终下载响应首部，即使它们不是输出的一部分。


# 重定向输入

传递请求数据的通用方法是通过重定向 `stdin`（标准输入）管道。这些数据被缓冲，然后没有进一步处理用作请求主体。使用管道有多种有用的方法：

从文件重定向：

``` bash
 $ http PUT example.com/person/1 X-API-Token:123 < person.json
```

或者另一个程序的输出：

``` bash
$ grep '401 Unauthorized' /var/log/httpd/error_log | http POST example.org/intruders
```

简单数据的数据你可以使用 `echo`：

``` bash
$ echo '{"name": "John"}' | http PATCH example.com/person/1 X-API-Token:123
```

您甚至可以使用 HTTPie 将 Web 服务连接在一起：

``` bash
$ http GET https://api.github.com/repos/jakubroztocil/httpie | http POST httpbin.org/post
```

您可以使用 `cat` 在终端上输入多行数据：

``` bash
$ cat | http POST example.com
<paste>
^D
```

``` bash
$ cat | http POST example.com/todos Content-Type:text/plain
- buy milk
- call parents
^D
```

在 OS X 上，您可以使用 `pbpaste` 发送剪贴板的内容：

``` bash
$ pbpaste | http PUT example.com
```

通过 `stdin` 传递数据不能与命令行中指定的数据字段组合：

``` bash
$ echo 'data' | http POST example.org more=data   # 这是无效的
```

要防止 HTTPie 读取 `stdin` 数据，可以使用 `--ignore-stdin` 选项。


## 从文件名中请求数据

重定向 `stdin` 的替代方法是指定一个文件名（比如 `@/path/to/file` ），其内容被用作来自 `stdin` 的文件。

它的优点是 `Content-Type` 首部根据文件扩展名自动设置为适当的值。例如，以下请求使用 `Content-Type: application/xml` 发送该 XML 文件的逐字内容：

``` bash
$ http PUT httpbin.org/put @/data/file.xml
```

# 终端输出

HTTPie 默认做了几件事，以使其终端输出易于阅读。

## 颜色和格式

语法突出显示应用于 HTTP 首部和正文（有意义的地方）。如果您不喜欢默认选项，可以通过 `--style` 选项选择首选颜色方案（有关可能的值，请参阅 `$ http --help`）。

此外，应用以下格式：

* HTTP 首部按名称排序。
* JSON 数据是缩进的，按键排序，unicode 转义转换为它们代表的字符。

其中一个选项可用于控制输出处理：

| - | - |
|:--------
| `--pretty=all` | 应用颜色和格式。终端输出的默认值。
| `--pretty=colors` | 应用颜色。
| `--pretty=format` | 应用格式。
| `--pretty=none` | 禁用输出处理。重定向输出的默认值。

## 二进制数据

终端输出禁止二进制数据，这样可以安全地对发送回二进制数据的 URL 执行请求。二进制数据在重定向时也被抑制，但是被美化输出。一旦我们知道响应主体是二进制的，连接就会关闭

``` bash
$ http example.org/Movie.mov
```

你几乎可以立即看到这样的东西：

``` bash
HTTP/1.1 200 OK
Accept-Ranges: bytes
Content-Encoding: gzip
Content-Type: video/quicktime
Transfer-Encoding: chunked

+-----------------------------------------+
| NOTE: binary data not shown in terminal |
+-----------------------------------------+
```

# 重定向输出

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


# 下载模式

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


## 下载的文件名

If not provided via ``--output, -o``, the output filename will be determined
from ``Content-Disposition`` (if available), or from the URL and
``Content-Type``. If the guessed filename already exists, HTTPie adds a unique
suffix to it.


## 下载时管道

You can also redirect the response body to another program while the response
headers and progress are still shown in the terminal:

.. code-block:: bash

    $ http -d https://github.com/jakubroztocil/httpie/archive/master.tar.gz |  tar zxf -



## 恢复下载

If ``--output, -o`` is specified, you can resume a partial download using the
``--continue, -c`` option. This only works with servers that support
``Range`` requests and ``206 Partial Content`` responses. If the server doesn't
support that, the whole file will simply be downloaded:

.. code-block:: bash

    $ http -dco file.zip example.org/file

## 其他说明

* The ``--download`` option only changes how the response body is treated.
* You can still set custom headers, use sessions, ``--verbose, -v``, etc.
* ``--download`` always implies ``--follow`` (redirects are followed).
* HTTPie exits with status code ``1`` (error) if the body hasn't been fully
  downloaded.
* ``Accept-Encoding`` cannot be set with ``--download``.


# 流式响应

Responses are downloaded and printed in chunks which allows for streaming
and large file downloads without using too much memory. However, when
`colors and formatting`_ is applied, the whole response is buffered and only
then processed at once.


## 禁用缓冲

You can use the ``--stream, -S`` flag to make two things happen:

1. The output is flushed in much smaller chunks without any buffering,
   which makes HTTPie behave kind of like ``tail -f`` for URLs.

2. Streaming becomes enabled even when the output is prettified: It will be
   applied to each line of the response and flushed immediately. This makes
   it possible to have a nice output for long-lived requests, such as one
   to the Twitter streaming API.


## 示例用例

Prettified streamed response:

.. code-block:: bash

    $ http --stream -f -a YOUR-TWITTER-NAME https://stream.twitter.com/1/statuses/filter.json track='Justin Bieber'


Streamed output by small chunks alá ``tail -f``:

.. code-block:: bash

    # Send each new tweet (JSON object) mentioning "Apple" to another
    # server as soon as it arrives from the Twitter streaming API:
    $ http --stream -f -a YOUR-TWITTER-NAME https://stream.twitter.com/1/statuses/filter.json track=Apple \
    | while read tweet; do echo "$tweet" | http POST example.org/tweets ; done

# Session

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