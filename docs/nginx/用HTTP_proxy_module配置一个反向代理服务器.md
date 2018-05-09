## 用 HTTP proxy module 配置一个反向代理服务器

反向代理（reverse proxy）方式是指用代理服务器来接受 Internet 上的连接请求，然后将请求转发给内部网络中的上游服务器，并将从上游服务器上得到的结果返回给 Internet上 请求连接的客户端，此时代理服务器对外的表现就是一个 Web 服务器。充当反向代理服务器也是 Nginx 的一种常见用法（反向代理服务器必须能够处理大量并发请求），本节将介绍 Nginx 作为 HTTP 反向代理服务器的基本用法。

由于 Nginx 具有“强悍”的高并发高负载能力，因此一般会作为前端的服务器直接向客户端提供静态文件服务。但也有一些复杂、多变的业务不适合放到 Nginx 服务器上，这时会用 Apache、Tomcat 等服务器来处理。于是，Nginx 通常会被配置为既是静态 Web 服务器也是反向代理服务器，不适合 Nginx 处理的请求就会直接转发到上游服务器中处理。

![](http://os6ycxx7w.bkt.clouddn.com/images/7defee24-a29e-4f77-be9a-a8f1286e448f.png)

与 Squid 等其他反向代理服务器相比，Nginx 的反向代理功能有自己的特点。

当客户端发来 HTTP 请求时，Nginx 并不会立刻转发到上游服务器，而是先把用户的请求（包括HTTP包体）完整地接收到 Nginx 所在服务器的硬盘或者内存中，然后再向上游服务器发起连接，把缓存的客户端请求转发到上游服务器。而 Squid 等代理服务器则采用一边接收客户端请求，一边转发到上游服务器的方式。

![](http://os6ycxx7w.bkt.clouddn.com/images/6f0872e7-20c0-4096-924d-c3eb00ef2183.png)

Nginx 的这种工作方式有什么优缺点呢？很明显，缺点是延长了一个请求的处理时间，并增加了用于缓存请求内容的内存和磁盘空间。而优点则是降低了上游服务器的负载，尽量把压力放在 Nginx 服务器上。

Nginx 的这种工作方式为什么会降低上游服务器的负载呢？通常，客户端与代理服务器之间的网络环境会比较复杂，多半是“走”公网，网速平均下来可能较慢，因此，一个请求可能要持续很久才能完成。而代理服务器与上游服务器之间一般是“走”内网，或者有专线连接，传输速度较快。Squid 等反向代理服务器在与客户端建立连接且还没有开始接收HTTP包体时，就已经向上游服务器建立了连接。例如，某个请求要上传一个1GB的文件，那么每次
 Squid 在收到一个TCP分包（如2KB）时，就会即时地向上游服务器转发。在接收客户端完整 HTTP 包体的漫长过程中，上游服务器始终要维持这个连接，这直接对上游服务器的并发处理能力提出了挑战。

Nginx 则不然，它在接收到完整的客户端请求（如1GB的文件）后，才会与上游服务器建立连接转发请求，由于是内网，所以这个转发过程会执行得很快。这样，一个客户端请求占用上游服务器的连接时间就会非常短，也就是说，Nginx 的这种反向代理方案主要是为了降低上游服务器的并发压力。

## 负载均衡的基本配置

作为代理服务器，一般都需要向上游服务器的集群转发请求。这里的负载均衡是指选择一种策略，尽量把请求平均地分布到每一台上游服务器上。下面介绍负载均衡的配置项。


### upstream 块

语法： `upstream name{...}`  
配置块： http

upstream 块定义了一个上游服务器的集群，便于反向代理中的 proxy_pass 使用。例如：

``` perl
upstream backend {
	server backend1.example.com;
	server backend2.example.com;
	server backend3.example.com;
}
server {
	location / {
		proxy_pass http://backend;
	}
}
```

### server

语法： `server name[parameters];`  
配置块： upstream

server 配置项指定了一台上游服务器的名字，这个名字可以是域名、IP 地址端口、UNIX 句柄等

``` perl
upstream backend {
	server backend1.example.com weight=5;
	server 127.0.0.1:8080 max_fails=3 fail_timeout=30s;
	server unix:/tmp/backend3;
	server a.example.com down;
	server b.example.com backup;
	server c.example.com max_conns=10000;
}
```

| 参数 | 含义  |
|:----|:------|
| down | 当前的 server 暂时不参与负载均衡
| backup | 预留的备份服务器，当其他所有服务器都不可用时，此服务器将被启用
| max_fails | 允许请求失败的次数
| fail_timeout | 经过 max_fails 失败后，服务暂停的时间
| max_conns |  限制最大接收的连接数



调度算法


| 参数 | 含义  |
|:----|:------|
| 默认 | 轮询，按时间顺序逐一分配到不同的服务器
| weight | 加权轮询， weight 值越大，分配到访问几率越高
| ip_hash | 每个请求按访问 IP 的 hash 结果分配， 这样来自同一个 IP 的固定访问一个服务器
| least_conn | 最少链接数， 哪个服务器连接数最少就连接哪个
| url_hash | 按照访问的 URL 的 hash 结果来分配请求，使每个 URL 定向到同一个服务器
| hash `key` | 自定义 hash 的 key

除了 weight 和其他参数配置方式一样（放到每一项的后面），其他调度参数都是单独一行的

``` perl
upstream backend {
	ip_hash;
	# least_conn;
	# url_hash;
	# hash $request_uri
	server a.example.com down;
	server b.example.com backup;
	server c.example.com max_conns=10000;
}
```


## 反向代理的基本配置

### proxy_pass

语法： `proxy_pass URL;`  
配置块： location、if

此配置项将当前请求反向代理到 URL 参数指定的服务器上，URL 可以是主机名或 IP 地址加端口的形式，例如：

```
proxy_pass http://localhost:8000/uri/;
```

还可以如上节负载均衡中所示，直接使用 upstream 块，例如：

```
upstream backend {
	…
}
server {
	location / {
		proxy_pass http://backend;
	}
}
```

用户可以把HTTP转换成更安全的HTTPS，例如：

```
proxy_pass https://192.168.0.1;
```

默认情况下反向代理是不会转发请求中的Host头部的。如果需要转发，那么必须加上配置：

```
proxy_set_header Host $host;
```

### 更多配置

``` perl
location / {
	proxy_pass http://backend;
	# proxy_redirect default;
	proxy_set_header Host $http_host;
	proxy_set_haader X-Real-IP $remote_addr;

	# proxy_connect_timeout 30;
	# proxy_send_timeout 60;
	# proxy_read_timeout 60;

	# proxy_buffer_size 32k;
	# proxy_buffering on;
	# proxy_buffers 4 128k;
	# proxy_busy_buffers_size 256k;
	# proxy_max_temp_file_size 256k;
}

```
!> 也可以把共用的配置放到单独的文件 `proxy_params` 中，然后通过 `include proxy_params` 引用。

## rewrite

### 应用场景
* URL 访问跳转，支持开发设计
 * 页面跳转、兼容支持、展示效果等
* SEO 优化
* 维护
 * 后台维护，流量转发等
* 安全

### 语法

Syntax: `rewrite regex replacement [flag];`  
Default: -  
Context: server, location, if  


### 应用

举个栗子

当网站需要维护的时候，需要将所有的请求转到一个静态 html，可以像下面这样做:

```
rewrite ^(.*)$ /pages/maintain.html break;
```

### flag

| flag | 含义  |
|:----|:------|
| last | 停止 rewrite 检测，能够跳转到其他 URI
| break | 停止 rewrite 检测，不能跳转到其他 URI
| redirect | 返回 302 临时重定向，地址栏会显示跳转后的地址
| permanent | 返回 301 永久重定向，地址栏会显示跳转后的地址。浏览器会记住重定向地址，以后直接访问重定向地址而不会再走服务器，除非清除缓存


last 和 break 的区别

``` perl
server {
	location ~ ^/break { 
		rewrite ^/break /test/ break; # 访问 /break 会 404 
		# rewrite ^/break test.html break; # 如果 test.html 存在可以访问
	}

	location ~ ^/last { # 访问 /last 可以跳转到 /test
		rewrite ^/last /test/ last;
	}

	location ~ ^/test/ {
		...
	}
}
```

## HTTPS

### 生成证书

需要安装 openssl 

```
openssl genrsa -idea -out jiuyou.key 1024
```
`-idea` 指明算法， `-out` 后面跟文件名，`1024` 代表位数

然后输入秘密，就会在目录下生成一个 `jiuyou.key` 文件


生成 csr 文件

```
openssl req -new -key jiuyou.key -out jiuyou.csr
```

生成证书（自签证书）

```
openssl x509 -req -days 3650 -in jiuyou.csr -signkey jiuyou.key -out jiuyou.crt
```
`-days` 代表签名过期时间

### 语法

Syntax: `ssl on | off;`  
Default: `ssl off;`  
Context: http, server  

Syntax: `ssl_certificate file;`  
Default: -  
Context: http, server  

Syntax: `ssl_certificate_key file;`  
Default: -  
Context: http, server  

### 应用

https 的默认端口是 443

``` perl
server {
	listen 443;
	server_name xxx.com;
	ssl on;
	ssl_certificate /path/jiuyou.crt;
	ssl_certificate_key /path/jiuyou.key;

	index index.html index.htm;

	location / {
		...
	}
}
```

### 苹果要求的证书

 1. 服务器所有连接使用 TLS1.2 以上的版本 (opensll 1.0.2)
 2. HTTPS 证书必须使用 SHA256 以上的 hash 算法签名
 3. HTTPS 证书必须使用 RSA 2048 位或 ECC 256 位以上公钥算法
 4. 使用前向加密技术



查看 openssl 版本

```
[root@localhost ~]# openssl version
OpenSSL 1.0.2k-fips  26 Jan 2017
```

查看证书的签名信息

```
openssl x509 -noout -text -in jiuyou.crt
```

生成符合苹果要求的证书

```
openssl req -days 3650 -x509 -sha256 -nodes -newkey rsa:2048 -keyout jiuyou.key -out jiuyou_apple.crt
```

!> 注意：以上命令会同时生成 `jiuyou.key` 和 `jiuyou_apple.crt` 文件

### 优化

``` perl
server {
	keepalive_timeout 100;

	ssl on;
	ssl_session_cache shared:SSL:10m; # 大约能缓存一万个左右的 session 会话
	ssl_session_timeout 10m;  # session 10分钟过期

	...
}
```