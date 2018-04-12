## 用 HTTP 核心模块配置一个静态 Web 服务器

除了基本配置项外，一个典型的静态 Web 服务器还会包含多个 server 块和 location 块，例如：

```
http {
	gzip on;
	upstream {
		...
	}
	...

	server {
		listen localhost:80;
		...
		
		location /webstatic {
			if ...{...}
			root optwebresource;
			...
		}
		location ~*.(jpg|jpeg|png|jpe|gif)$ {
			...
		}
	}
	server {
		...
	}
}
```

Nginx 为配置一个完整的静态 Web 服务器提供了非常多的功能，下面会把这些配置项分为以下 8 类进行详述：虚拟主机与请求的分发、文件路径的定义、内存及磁盘资源的分配、网络连接的设置、MIME 类型的设置、对客户端请求的限制、文件操作的优化、对客户端请求的特殊处理。这种划分只是为了帮助大家从功能上理解这些配置项。


## 虚拟主机与请求的分发

由于 IP 地址的数量有限，因此经常存在多个主机域名对应着同一个 IP 地址的情况，这时在 nginx.conf 中就可以按照 server_name（对应用户请求中的主机域名）并通过 server 块来定义虚拟主机，每个 server 块就是一个虚拟主机，它只处理与之相对应的主机域名请求。这样，一台服务器上的 Nginx 就能以不同的方式处理访问不同主机域名的 HTTP 请求了。

### 监听端口

语法： `listen address:port[default(deprecated in 0.8.21)|default_server|[backlog=num|rcvbuf=size|sndbuf=size|accept_filter=filter|deferred|bind|ipv6only=[on|off]|ssl]];`  
默认： `listen 80;`  
配置块： `server`

`listen` 参数决定 Nginx 服务如何监听端口。在 listen 后可以只加 IP 地址、端口或主机名，非常灵活，例如：

```
listen 127.0.0.1:8000;
listen 127.0.0.1; # 注意：不加端口时，默认监听 80 端口
listen 8000;
listen *:8000;
listen localhost:8000;
```

如果服务器使用 IPv6 地址，那么可以这样使用：

```
listen [::]:8000;
listen [fe80::1];
listen [:::a8c9:1234]:80;
```

在地址和端口后，还可以加上其他参数，例如：

```
listen 443 default_server ssl;
listen 127.0.0.1 default_server accept_filter=dataready backlog=1024;
```

listen 可用参数的意义。  
 * `default` -- 将所在的 server 块作为整个 Web 服务的默认 server 块。如果没有设置这个参数，那么将会以在 nginx.conf 中找到的第一个 server 块作为默认 server 块。为什么需要默认虚拟主机呢？当一个请求无法匹配配置文件中的所有主机域名时，就会选用默认的虚拟主机。
 * `default_server` -- 同上。
 * `bind`  -- 绑定当前端口/地址对，如 127.0.0.1:8000。只有同时对一个端口监听多个地址时才会生效。
 * `ssl`  -- 在当前监听的端口上建立的连接必须基于 SSL 协议。

### 主机名称

语法： `server_name name[...];`  
默认： `server_name "";`  
配置块： `server`  

server_name 后可以跟多个主机名称，如  server_name www.testweb.com 、download.testweb.com;。

在开始处理一个 HTTP 请求时，Nginx 会取出 header 头中的 Host，与每个 server 中的 server_name 进行匹配，以此决定到底由哪一个 server 块来处理这个请求。有可能一个 Host 与多个 server 块中的  server_name 都匹配，这时就会根据匹配优先级来选择实际处理的 server 块。server_name 与 Host 的匹配优先级如下：

 1. 首先选择所有字符串完全匹配的 server_name，如 `www.testweb.com`。
 2. 其次选择通配符在前面的 server_name，如 `*.testweb.com`。
 3. 再次选择通配符在后面的 server_name，如 `www.testweb.*`。
 4. 最后选择使用正则表达式才匹配的 server_name，如 `~^\.testweb\.com$`。

!> Nginx 正是使用 server_name 配置项针对特定 Host 域名的请求提供不同的服务，以此实现虚拟主机功能。


### location

语法： `location [=|~|~*|^~|@]/uri/{...}`  
配置块： `server`

location 会尝试根据用户请求中的 URI 来匹配上面的 `/uri` 表达式，如果可以匹配，就选择 `location{}` 块中的配置来处理用户请求。当然，匹配方式是多样的，下面介绍 location 的匹配规则。

 1. `=` 表示把 URI 作为字符串，以便与参数中的 uri 做完全匹配。例如：
```
 location = / { # 只有当用户请求是 / 时，才会使用该 location 下的配置
	 ...
 }
```
 2. `~` 表示匹配 URI 时是字母大小写敏感的。
 3. `~*` 表示匹配 URI 时忽略字母大小写问题。
 4. `^~` 表示匹配 URI 时只需要其前半部分与 uri 参数匹配即可。例如：
```
location ^~ images { # 以 images 开始的请求都会匹配上
	...
}
```
 5. `@` 表示仅用于 Nginx 服务内部请求之间的重定向，带有 `@` 的 location 不直接处理用户请求。


当然，在 uri 参数里是可以用正则表达式的，例如：

```
location ~* \.(gif|jpg|jpeg)$ {  # 匹配以 .gif、.jpg、.jpeg 结尾的请求
	...
}
```

!> 注意，location 是有顺序的，当一个请求有可能匹配多个 location 时，实际上这个请求会被第一个 location 处理。



在以上各种匹配方式中，都只能表达为“如果匹配...则...”。如果需要表达“如果不匹配...则...”，就很难直接做到。有一种解决方法是在最后一个 location 中使用 / 作为参数，它会匹配所有的 HTTP 请求，这样就可以表示如果不能匹配前面的所有 location，则由 “/” 这个 location 处理。例如：

```
location / {  # / 可以匹配所有请求
	...
}
```

## 文件路径的定义

### 以 root 方式设置资源路径

语法： `root path;`  
默认： `root html;`  
配置块： `http`、`server`、`location`、`if`

例如，定义资源文件相对于HTTP请求的根目录。

```
location /download/ {
	root /optwebhtml;
}
```

在上面的配置中，如果有一个请求的 URI 是 `/download/index/test.html`，那么 Web 服务器将会返回服务器上 `/optwebhtml/download/index/test.html` 文件的内容。

### 以 alias 方式设置资源路径

语法： `alias path;`  
配置块： location


`alias` 也是用来设置文件资源路径的，它与 `root` 的不同点主要在于如何解读紧跟 `location` 后面的 uri 参数，这将会致使 `alias` 与 `root` 以不同的方式将用户请求映射到真正的磁盘文件上。例如，如果有一个请求的 URI 是 `/conf/nginx.conf`，而用户实际想访问的文件在 `/usr/local/nginx/conf/nginx.conf`，那么想要使用 `alias` 来进行设置的话，可以采用如下方式：

```
location conf {
	alias /usr/local/nginx/conf/;
}
```

如果用 root 设置，那么语句如下所示：

```
location conf {
	root /usr/local/nginx/;
}
```

使用 `alias` 时，在 URI 向实际文件路径的映射过程中，已经把 `location` 后配置的 `/conf` 这部分字符串丢弃掉，因此，`/conf/nginx.conf` 请求将根据 `alias path` 映射为 `/path/nginx.conf`。`root` 则不然，它会根据完整的 URI 请求来映射，因此，`/conf/nginx.conf` 请求会根据 `root path` 映射为 `/path/conf/nginx.conf`。这也是 `root` 可以放置到 http、server、location 或 if 块中，而 `alias` 只能放置到 location 块中的原因。

`alias` 后面还可以添加正则表达式，例如：

```
location ~ ^/test/(\w+)\.(\w+)$ {
	alias /usr/local/nginx/$2/$1.$2;
}
```

这样，请求在访问 `/test/nginx.conf`时，Nginx会返回 `/usr/local/nginx/conf/nginx.conf`文件中的内容。

### 访问首页

语法： `index file...;`  
默认： `index index.html;`  
配置块： http、server、location

有时，访问站点时的 URI 是 `/`，这时一般是返回网站的首页，而这与 root 和 alias 都不同。这里用 ngx_http_index_module 模块提供的 index 配置实现。index 后可以跟多个文件参数，Nginx 将会按照顺序来访问这些文件，例如：

```
location / {
	root path;
	index index.html htmlindex.php /index.php;
}
```

### 根据 HTTP 返回码重定向页面

语法： `error_page code[code...][=|=answer-code]uri|@named_location`  
配置块： http、server、location、if

当对于某个请求返回错误码时，如果匹配上了 `error_page` 中设置的 code，则重定向到新
的 URI 中。例如：

```
error_page 404 404.html;
error_page 502 503 504 50x.html;
error_page 403 http://example.com/forbidden.html;
error_page 404 = @fetch;
```

注意，虽然重定向了 URI，但返回的 HTTP 错误码还是与原来的相同。用户可以通过 “=” 来更改返回的错误码，例如：

```
error_page 404 =200 empty.gif;
error_page 404 =403 forbidden.gif;
```

也可以不指定确切的返回错误码，而是由重定向后实际处理的真实结果来决定，这时，只要把“=”后面的错误码去掉即可，例如：

```
error_page 404 = /empty.gif;
```

如果不想修改 URI，只是想让这样的请求重定向到另一个 location 中进行处理，那么可以这样设置：

```
location / (
	error_page 404 @fallback;
)
location @fallback (
	proxy_pass http://backend;
)
```

这样，返回 404 的请求会被反向代理到 http://backend 上游服务器中处理。

## 内存及磁盘资源的分配


### HTTP 包体只存储到磁盘文件中

语法： `client_body_in_file_only on|clean|off;`  
默认： `client_body_in_file_only off;`  
配置块： http、server、location

### HTTP 包体尽量写入到一个内存 buffer 中

语法： `client_body_in_single_buffer on|off;`  
默认： `client_body_in_single_buffer off;`  
配置块： http、server、location

### 存储 HTTP 头部的内存 buffer 大小

语法： `client_header_buffer_size size;`  
默认： `client_header_buffer_size 1k;`  
配置块： http、server

### 存储超大 HTTP 头部的内存 buffer 大小

语法： `large_client_header_buffers number size;`  
默认： `large_client_header_buffers 48k;`  
配置块： http、server

### 存储 HTTP 包体的内存 buffer 大小

语法： `client_body_buffer_size size;`  
默认： `client_body_buffer_size 8k/16k;`  
配置块： http、server、location

### HTTP 包体的临时存放目录

语法： `client_body_temp_path dir-path[level1[level2[level3]]];`  
默认： `client_body_temp_path client_body_temp;`  
配置块： http、server、location


### connection_pool_size

语法： `connection_pool_size size;`  
默认： `connection_pool_size 256;`  
配置块： http、server

### request_pool_size

语法： `request_pool_size size;`  
默认： `request_pool_size 4k;`   
配置块： http、server

## 网络连接的设置

### 读取 HTTP 头部的超时时间

语法： `client_header_timeout time（默认单位：秒）;`  
默认： `client_header_timeout 60;`  
配置块： http、server、location

### 读取 HTTP 包体的超时时间

语法： `client_body_timeout time（默认单位：秒）;`  
默认： `client_body_timeout 60;`  
配置块： http、server、location

### 发送响应的超时时间

语法： `send_timeout time;`  
默认： `send_timeout 60;`  
配置块： http、server、location

### reset_timeout_connection

语法： `reset_timeout_connection on|off;`  
默认： `reset_timeout_connection off;`  
配置块： http、server、location

### lingering_close

语法： `lingering_close off|on|always;`  
默认： `lingering_close on;`  
配置块： http、server、location

### lingering_time

语法： `lingering_time time;`  
默认： `lingering_time 30s;`  
配置块： http、server、location

### lingering_timeout

语法： `lingering_timeout time;`  
默认： `lingering_timeout 5s;`  
配置块： http、server、location

### 对某些浏览器禁用 keepalive 功能

语法： `keepalive_disable [msie6|safari|none]...`  
默认： `keepalive_disable msie6 safari`  
配置块： http、server、location

HTTP 请求中的 keepalive 功能是为了让多个请求复用一个 HTTP 长连接，这个功能对服务器的性能提高是很有帮助的。但有些浏览器，如 IE 6 和 Safari，它们对于使用 keepalive 功能的 POST 请求处理有功能性问题。因此，针对 IE 6 及其早期版本、 Safari 浏览器默认是禁用 keepalive 功能的。

### keepalive 超时时间

语法： `keepalive_timeout time（默认单位：秒）;`  
默认： `keepalive_timeout 75;`  
配置块： http、server、location


一个 keepalive 连接在闲置超过一定时间后（默认的是75秒），服务器和浏览器都会去关闭这个连接。当然，keepalive_timeout 配置项是用来约束 Nginx 服务器的，Nginx 也会按照规范把这个时间传给浏览器，但每个浏览器对待 keepalive 的策略有可能是不同的。

### 一个 keepalive 长连接上允许承载的请求最大数

语法： `keepalive_requests n;`  
默认： `keepalive_requests 100;`  
配置块： http、server、location


### tcp_nodelay

语法： `tcp_nodelay on|off;`  
默认： `tcp_nodelay on;`  
配置块： http、server、location


确定对 keepalive 连接是否使用 TCP_NODELAY 选项。


## MIME 类型的设置

### MIME type 与文件扩展的映射

语法： `type{...};`  
配置块： http、server、location

定义 MIME type 到文件扩展名的映射。多个扩展名可以映射到同一个 MIME type。例如：

```
types {
	text/html html;
	text/html conf;
	image/gif gif;
	image/jpeg jpg;
}
```

### 默认 MIME type

语法： `default_type MIME-type;`  
默认： `default_type text/plain;`  
配置块： http、server、location

当找不到相应的 MIME type 与文件扩展名之间的映射时，使用默认的 MIME type 作为 HTTP header 中的 Content-Type。


### types_hash_bucket_size

语法： `types_hash_bucket_size size;`  
默认： `types_hash_bucket_size 32|64|128;`  
配置块： http、server、location


### types_hash_max_size

语法： `types_hash_max_size size;`  
默认： `types_hash_max_size 1024;`  
配置块： http、server、location


## 对客户端请求的限制

### 按 HTTP 方法名限制用户请求

语法： `limit_except method...{...}`  
配置块： location

Nginx 通过 limit_except 后面指定的方法名来限制用户请求。方法名可取值包括：GET、HEAD、POST、PUT、DELETE、MKCOL、COPY、MOVE、OPTIONS、PROPFIND、PROPPATCH、LOCK、UNLOCK 或者 PATCH。例如：

```
limit_except GET {
	allow 192.168.1.0/32;
	deny all;
}
```

### HTTP 请求包体的最大值

语法： `client_max_body_size size;`  
默认： `client_max_body_size 1m;`  
配置块： http、server、location

### 对请求的限速

语法： `limit_rate speed;`  
默认： `limit_rate 0;`  
配置块： http、server、location、if

此配置是对客户端请求限制每秒传输的字节数。

针对不同的客户端，可以用 $limit_rate 参数执行不同的限速策略。例如：

```
server {
	if ($slow) {
		set $limit_rate 4k;
	}
}
```

### limit_rate_after

语法： `limit_rate_after time;`  
默认： `limit_rate_after 1m;`  
配置块： http、server、location、if

此配置表示 Nginx 向客户端发送的响应长度超过 limit_rate_after 后才开始限速。例如：

```
limit_rate_after 1m;
limit_rate 100k;
```



## 文件操作的优化

### sendfile 系统调用

语法： `sendfile on|off;`  
默认： `sendfile off;`  
配置块： http、server、location

可以启用 Linux 上的 sendfile 系统调用来发送文件，它减少了内核态与用户态之间的两次内存复制，这样就会从磁盘中读取文件后直接在内核态发送到网卡设备，提高了发送文件的效率。


### AIO系统调用

语法： `aio on|off;`  
默认： `aio off;`  
配置块： http、server、location


### directio

语法： `directio size|off;`  
默认： `directio off;`  
配置块： http、server、location


### directio_alignment

语法： `directio_alignment size;`  
默认： `directio_alignment 512;`  
配置块： http、server、location

### 打开文件缓存

语法： `open_file_cache max=N[inactive=time]|off;`  
默认： `open_file_cache off;`  
配置块： http、server、location

文件缓存会在内存中存储以下3种信息：
 * 文件句柄、文件大小和上次修改时间。
 * 已经打开过的目录结构。
 * 没有找到的或者没有权限操作的文件信息。

这样，通过读取缓存就减少了对磁盘的操作。

该配置项后面跟3种参数。
 * `max`  --  表示在内存中存储元素的最大个数。当达到最大限制数量后，将采用 LRU（Least Recently Used）算法从缓存中淘汰最近最少使用的元素。
 * `inactive`  --  表示在 inactive 指定的时间段内没有被访问过的元素将会被淘汰。默认时间为60秒。
 * `off`  --  关闭缓存功能。

例如：

```
open_file_cache max=1000 inactive=20s;
```

### 是否缓存打开文件错误的信息

语法： `open_file_cache_errors on|off;`  
默认： `open_file_cache_errors off;`  
配置块： http、server、location

### 不被淘汰的最小访问次数

语法： `open_file_cache_min_uses number;`  
默认： `open_file_cache_min_uses 1;`  
配置块： http、server、location


### 检验缓存中元素有效性的频率

语法： `open_file_cache_valid time;`  
默认： `open_file_cache_valid 60s;`  
配置块： http、server、location


## 对客户端请求的特殊处理

### 忽略不合法的 HTTP 头部

语法： `ignore_invalid_headers on|off;`
默认： `ignore_invalid_headers on;`
配置块： http、server

如果将其设置为 off，那么当出现不合法的 HTTP 头部时，Nginx 会拒绝服务，并直接向用户发送 400（Bad Request）错误。如果将其设置为 on，则会忽略此 HTTP 头部。

### HTTP 头部是否允许下划线

语法： `underscores_in_headers on|off;`  
默认： `underscores_in_headers off;`  
配置块： http、server


### 对 If-Modified-Since 头部的处理策略

语法： `if_modified_since[off|exact|before];`  
默认： `if_modified_since exact;`  
配置块： http、server、location

### 文件未找到时是否记录到 error 日志

语法： `log_not_found on|off;`  
默认： `log_not_found on;`  
配置块： http、server、location


### merge_slashes
语法： `merge_slashes on|off;`  
默认： `merge_slashes on;`  
配置块： http、server、location

此配置项表示是否合并相邻的 “/”，例如，`//test///a.txt`，在配置为 on 时，会将其匹配为 `location/test/a.txt`；如果配置为 off，则不会匹配，URI 将仍然是 `//test///a.txt`。

### DNS 解析地址

语法： `resolver address...;`  
配置块： http、server、location

设置 DNS 名字解析服务器的地址，例如：

```
resolver 127.0.0.1 192.0.2.1;
```

### DNS 解析的超时时间

语法： `resolver_timeout time;`  
默认： `resolver_timeout 30s;`  
配置块： http、server、location

### 返回错误页面时是否在 Server 中注明 Nginx 版本

语法： `server_tokens on|off;`  
默认： `server_tokens on;`  
配置块： http、server、location