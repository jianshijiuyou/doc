> [原文链接:使用uWSGI和nginx来设置Django和你的web服务器](http://uwsgi-docs-zh.readthedocs.io/zh_CN/latest/tutorials/Django_and_nginx.html)

## Django + uWSGI + Nginx

## 基本的 uWSGI 安装和配置

### 把 uWSGI 安装到你的 virtualenv 中

```
pip install uwsgi
```

!> 记住，你将需要安装 Python 开发包。对于 Debian，或者 Debian 衍生系统，例如 Ubuntu，你需要安装的是  pythonX.Y-dev ，其中，X.Y 是你 Python 的版本。

### 基础测试

创建一个名为 `test.py` 文件:

``` python
# test.py
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return [b"Hello World"] # python3
    #return ["Hello World"] # python2
```

运行 uWSGI:

```
uwsgi --http :8000 --wsgi-file test.py
```

选项表示:
 * `http :8000`: 使用 http 协议，端口8000
 * `wsgi-file test.py`: 加载指定的文件，test.py


当浏览器访问 8000 端口时，这将直接提供一个’hello world’消息。 访问:

```
http://example.com:8000
```

来看一看。如果是这样，那么意味着以下的组件栈正常:

```
the web client <-> uWSGI <-> Python
```

## 测试你的 Django 项目

现在，我们想让 uWSGI 做同样的事，但是返回一个 Django 站点而不是 test.py 模块。

请确保你的 mysite 项目实际上正常工作:

```
python manage.py runserver 0.0.0.0:8000
```

而如果正常，则使用 uWSGI 来运行它:

```
uwsgi --http :8000 --module mysite.wsgi
```
 * `module mysite.wsgi`:  加载指定的 wsgi 模块


将你的浏览器指向该服务器；如果站点出现，那么意味着 uWSGI 可以为你虚拟环境中的 Django 应用服务，而这个栈工作正常:

```
the web client <-> uWSGI <-> Django
```

现在，通常我们不会让浏览器直接与 uWSGI 通信。那是 web 服务器的工作，这是个穿针引线的活。

## 为你的站点配置 nginx

你会需要 `uwsgi_params` 文件，不过，一般在你的 nginx.conf 同级目录下该文件已经存在，如果不存在，可以从 [https://github.com/nginx/nginx/blob/master/conf/uwsgi_params](https://github.com/nginx/nginx/blob/master/conf/uwsgi_params) 获取。

修改 nginx.conf 文件

```
# mysite_nginx.conf

# the upstream component nginx needs to connect to
upstream django {
    # server unix:///path/to/your/mysite/mysite.sock; # for a file socket
    server 127.0.0.1:8001; # for a web port socket (we'll use this first)
}

# configuration of the server
server {
    # the port your site will be served on
    listen      8000;
    # the domain name it will serve for
    server_name .example.com; # substitute your machine's IP address or FQDN
    charset     utf-8;

    # max upload size
    client_max_body_size 75M;   # adjust to taste

    # Django media
    location /media  {
        alias /path/to/your/mysite/media;  # your Django project's media files - amend as required
    }

    location /static {
        alias /path/to/your/mysite/static; # your Django project's static files - amend as required
    }

    # Finally, send all non-media requests to the Django server.
    location / {
        uwsgi_pass  django;
        include     uwsgi_params; # uwsgi_params 文件路径，当前配置说明它和 nginx.conf 文件同级目录
    }
}
```

这个配置文件告诉 nginx 提供来自文件系统的媒体和静态文件，以及处理那些需要 Django 干预的请求。对于一个大型部署，让一台服务器处理静态/媒体文件，让另一台处理 Django 应用，被认为是一种很好的做法，但是现在，这样就好了。

## 部署静态文件

在运行 nginx 之前，你必须收集所有的 Django 静态文件到静态文件夹里。首先，必须编辑 mysite/settings.py，添加:

``` python
#注释掉下面这个配置
#STATICFILES_DIRS = [
#    os.path.join(BASE_DIR, "static")
#]

STATIC_ROOT = os.path.join(BASE_DIR, "static")
```

然后运行

``` python
python manage.py collectstatic
```
这条命令会把所有用到的静态文件都复制到 static 目录下

## 基本的 nginx 测试

使运行中的 Nginx 重读配置项并生效

```
/usr/local/nginx/sbin/nginx -s reload
```

要检查是否正确的提供了媒体文件服务，添加一个名为 `media.png` 的图像到 `/path/to/your/project/project/media directory` 中，然后访问 `http://example.com:8000/media/media.png` - 如果这能正常工作，那么至少你知道 nginx 正在正确的提供文件服务。

## nginx 和 uWSGI 以及 test.py

让 nginx 对 `test.py` 应用说句 ”hello world” 吧。

```
uwsgi --socket :8001 --wsgi-file test.py
```

这几乎与之前相同，除了这次有一个选项不同：
 * `socket :8001` : 使用 uwsgi 协议，端口为8001

同时，已经配置了 nginx 在那个端口与 uWSGI 通信，而对外使用 8000 端口。访问:
```
http://example.com:8000/
```

来检查。而这是我们的栈:

```
the web client <-> the web server <-> the socket <-> uWSGI <-> Python
```

## 使用 Unix socket 而不是端口

目前，我们使用了一个TCP端口 socket，因为它简单些，但事实上，使用 Unix socket 会比端口更好 - 开销更少。

编辑 `nginx.conf`, 修改它以匹配:

```
server unix:///path/to/your/mysite/mysite.sock; # for a file socket
# server 127.0.0.1:8001; # for a web port socket (we'll use this first)
```

然后重启nginx。

再次运行uWSGI:

```
uwsgi --socket mysite.sock --wsgi-file test.py
```

这次， `socket` 选项告诉 uWSGI 使用哪个文件。

在浏览器中尝试访问 `http://example.com:8000/`。

### 如果那不行

检查 nginx 错误日志 。如果你看到像这样的信息:
```
connect() to unix:///path/to/your/mysite/mysite.sock failed (13: Permission
denied)
```
那么可能你需要管理这个socket上的权限，从而允许 nginx 使用它。

尝试:

```
uwsgi --socket mysite.sock --wsgi-file test.py --chmod-socket=666 # (very permissive)
or
uwsgi --socket mysite.sock --wsgi-file test.py --chmod-socket=664 # (more sensible)
```

你可能还必须添加你的用户到 nginx 的组 (可能是 www-data)，反之亦然，这样，nginx 可以正确地读取或写入你的socket。

## 使用 uwsgi 和 nginx 运行 Django 应用

运行我们的Django应用：

```
uwsgi --socket mysite.sock --module mysite.wsgi --chmod-socket=664
```

现在，uWSGI 和 nginx 应该不仅仅可以为一个”Hello World”模块服务，还可以为你的 Django 项目服务。

## 配置 uWSGI 以允许 .ini 文件

我们可以将用在 uWSGI 上的相同的选项放到一个文件中，然后告诉 uWSGI 使用该文件运行。这使得管理配置更容易。

创建一个名为 `mysite_uwsgi.ini` 的文件:

```
# mysite_uwsgi.ini file
[uwsgi]

# Django-related settings
# the base directory (full path)
chdir           = /path/to/your/project
# Django's wsgi file
module          = project.wsgi
# the virtualenv (full path)
home            = /path/to/virtualenv

# process-related settings
# master
master          = true
# maximum number of worker processes
processes       = 10
# the socket (use the full path to be safe
socket          = /path/to/your/project/mysite.sock
# ... with appropriate permissions - may be needed
# chmod-socket    = 664
# clear environment on exit
vacuum          = true
```

然后使用这个文件运行uswgi：

```
uwsgi --ini mysite_uwsgi.ini # the --ini option is used to specify a file
```

再次，测试Django站点是否如预期工作。


## 系统级安装uWSGI

目前，uWSGI只装在我们的虚拟环境中；出于部署需要，我们将需要让它安装在系统范围中。

停用你的虚拟环境:

```
deactivate
```

然后在系统范围中安装uWSGI:

```
sudo pip install uwsgi
```

再次检查你是否仍然能如之前那样运行uWSGI:

```
uwsgi --ini mysite_uwsgi.ini # the --ini option is used to specify a file
```