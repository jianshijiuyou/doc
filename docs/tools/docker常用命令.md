> [总结自：Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice/content/)

## 访问仓库

| 命令 | 含义 |
| :-------|:---------
| `docker login` | 命令交互式登陆 
| `docker logout` | 退出登录 
| `docker search` | 查找官方仓库中的镜像 
| `docker pull` | 下载镜像到本地
| `docker push` | 将自己的镜像推送到 Docker Hub <br>例：`docker push username/ubuntu:17.10`


## 镜像管理

### 列出镜像
```
[root@localhost ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              e38bc07ac18e        3 weeks ago         1.85kB
centos              latest              e934aafc2206        4 weeks ago         199MB

```
列表包含了 `仓库名`、`标签`、`镜像 ID`、`创建时间` 以及 `所占用的空间`。

默认的 `docker image ls` 列表中只会显示顶层镜像。如果希望显示包括中间层镜像在内的所有镜像的话，需要加 `-a` 参数。

```
$ docker image ls -a
```

根据仓库名列出镜像

```
$ docker image ls ubuntu
```

列出特定的某个镜像，也就是说指定仓库名和标签

```
$ docker image ls ubuntu:16.04
```

除此以外，`docker image ls` 还支持强大的过滤器参数 `--filter`，或者简写 `-f`。

通过 `-f` 列出虚悬镜像

```
$ docker image ls -f dangling=true
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              00285df0df87        5 days ago          342 MB
```
!> 删除虚悬镜像用 `docker image prune`

只显示镜像的 ID

```
$ docker image ls -q
```

利用 `--format` 以特定格式显示

比如，只包含镜像 ID 和仓库名：

```
$ docker image ls --format "{{.ID}}: {{.Repository}}"
```

或者打算以表格等距显示，并且有标题行，和默认一样，不过自己定义列：

```
$ docker image ls --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
```

### 删除镜像

```
$ docker image rm [选项] <镜像1> [<镜像2> ...]
```

其中，`<镜像>` 可以是 `镜像短 ID`、`镜像长 ID`、`镜像名` 或者 `镜像摘要`。

通过 `镜像短ID` 删除
```
$ docker image rm e38
```

通过 `<仓库名>:<标签>` 删除

```
$ docker image rm centos
```

通过 `镜像摘要` 删除。

```
$ docker image ls --digests
REPOSITORY                  TAG                 DIGEST                                                                    IMAGE ID            CREATED             SIZE
node                        slim                sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228   6e0c4c8e3913        3 weeks ago         214 MB

$ docker image rm node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228
Untagged: node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228
```

结合 `docker image ls -q` 进行批量删除

例：删除所有仓库名为 `redis` 的镜像：

```
$ docker image rm $(docker image ls -q redis)
```

删除所有在 `mongo:3.2` 之前的镜像：

```
$ docker image rm $(docker image ls -q -f before=mongo:3.2)
```

### history

可以用 `docker history` 具体查看镜像内的历史记录


### 构建镜像

```
docker build [选项] <上下文路径/URL/->

# 举个栗子

docker build -t nginx:v3 .
```

* `-t` 指定一个以 'name:tag' 格式的命名
* `-f` 指定 Dockerfile 文件路径（名字不一定是 `Dockerfile`）

如果没有指定 `-f`，那么会在上下文路径中寻找 `Dockerfile` 文件






## 容器管理

### 启动

命令主要为 `docker run`。

下面的命令输出一个 “Hello World”，之后终止容器。

```
$ docker run ubuntu:14.04 /bin/echo 'Hello world'
Hello world
```

下面的命令则启动一个 bash 终端，允许用户进行交互。
```
$ docker run -t -i ubuntu:14.04 /bin/bash
root@af8bae53bdd3:/#
```

* `-t`: 让 Docker 分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上
* `-i`: 让容器的标准输入保持打开

通过 `--name` 指定容器的名称

```
[root@localhost ~]# docker run --name test centos /bin/echo "hello"
hello
[root@localhost ~]# docker container ls -a --format "{{.ID}} - {{.Names}}"
041b77b8845e - test
```

#### 重启政策（--restart）

使用 Docker 的 `--restart` 指定容器的重启策略。重启策略控制 Docker 守护进程在退出后是否重新启动容器。 Docker 支持以下重启策略：

| 策略 | 结果 |
|:------------
| `no` | 退出时不要自动重启容器。这是默认值。
| `on-failure[:max-retries]` | 仅当容器以非零退出状态退出时才重新启动。 限制 Docker 守护进程尝试的重新启动重试次数（可选）。
| `unless-stopped` | 除非显式停止或 Docker 本身重启或停止，否则重新启动容器。
| `always` | 无论退出状态如何，始终重新启动容器。当您指定 `always` 时，Docker 守护进程将尝试无限期地重新启动容器。无论容器的当前状态如何，容器也将始终在守护进程启动时启动。

```
$ docker run --restart=always redis
```

这将运行带有重启策略的 redis 容器，这样如果容器退出，Docker 将重新启动它。


### 后台运行
通过添加 `-d` 参数来实现。

```
[root@localhost ~]# docker run -d centos /bin/sh -c "while true; do echo hello world; sleep 1; done"
f0a77b17adf367f97f4043d83e3645a7c78e3f9b785cd4626122877980626b15
```
要获取容器的输出信息，可以通过 `docker container logs [container ID or NAMES]` 命令。

```
[root@localhost ~]# docker container logs f0a77b17adf3
hello world
hello world
hello world
```

### 终止容器

可以使用 `docker container stop` 来终止一个运行中的容器。

此外，当 Docker 容器中指定的应用终结时，容器也自动终止。

```
docker container stop f0a77b17adf3
```

处于终止状态的容器，可以通过 `docker container start ...` 命令来重新启动。

此外，`docker container restart ...` 命令会将一个运行态的容器终止，然后再重新启动它。


### 进入容器

* 通过 `attach`

```
[root@localhost ~]# docker run -dit centos
11cf2828d1522c3dbc93a665e1bb7eb236944fe250b66945e76946be67a1cfc1

[root@localhost ~]# docker attach 11cf
[root@11cf2828d152 /]#
```

!> 注意：如果从这个 stdin 中 exit，会导致容器的停止。

* 通过 `exec` (推荐)

`docker exec` 后边可以跟多个参数，这里主要说明 `-i` `-t` 参数。

只用 `-i` 参数时，由于没有分配伪终端，界面没有我们熟悉的 Linux 命令提示符，但命令执行结果仍然可以返回。

当 `-i` `-t` 参数一起使用时，则可以看到我们熟悉的 Linux 命令提示符。

```
[root@localhost ~]# docker exec -i <name or id> bash
# or
[root@localhost ~]# docker exec -it <name or id> bash
```

!> 如果从这个 stdin 中 exit，不会导致容器的停止。


### 导出导入快照

导出

```
$ docker export 7691a814370e > ubuntu.tar
```

导入

```
$ cat ubuntu.tar | docker import - test/ubuntu:v1.0
```

### 删除

```
[root@localhost ~]# docker container rm 3842866f70df
```
如果要删除一个运行中的容器，可以添加 `-f` 参数。

清理所有处于终止状态的容器

```
$ docker container prune
```

### diff

查看容器的改动

```
docker diff [ID | name]
```

### 查看容器的信息

```
docker inspect [container name]
```

### 查看日志

```
docker logs [OPTIONS] CONTAINER
```

OPTIONS 说明：

* `-f` : 跟踪日志输出
* `--since` : 显示某个开始时间的所有日志
* `-t` : 显示时间戳
* `--tail` : 仅列出最新 N 条容器日志

栗子

```
docker logs -f mynginx

docker logs --since="2016-07-01" --tail=10 mynginx
```

### ps

```
docker ps [OPTIONS]
```

OPTIONS 说明：

* `-a`: 显示所有的容器，包括未运行的。
* `-f`: 根据条件过滤显示的内容。
* `--format`: 指定返回值的模板文件。
* `-l`: 显示最近创建的容器。
* `-n`: 列出最近创建的n个容器。
* `--no-trunc`: 不截断输出。
* `-q`: 静默模式，只显示容器编号。
* `-s`: 显示总的文件大小。

#### Filtering

过滤标志（ `-f` 或 `--filter` ）格式是 `key=value` 对。如果有多个过滤器，则传递多个标志（例如 `--filter "foo=bar" --filter "bif=baz"` ）

常用过滤器有：

| Filter | Description |
|:---------------------------------
| id	    | Container’s ID
| name	    | Container’s name
| label	    | 表示键或键值对的任意字符串。表示为 `<key>` 或 `<key>=<value>`
| exited	| 表示容器退出代码的整数。仅适用于 `--all` 。
| status	| One of `created`, `restarting`, `running`, `removing`, `paused`, `exited`, or `dead`
| volume	| 过滤运行已安装给定卷或绑定装载的容器。
| network	| 过滤运行连接到给定网络的容器。
| publish or expose	| 过滤发布或公开给定端口的容器。 表示为 `<port>[/<proto>]` or `<startport-endport>/[<proto>]`
| health	| 根据容器的健康状况筛选容器. One of `starting`, `healthy`, `unhealthy` or `none`.

更多过滤器和示例见 [filtering](https://docs.docker.com/engine/reference/commandline/ps/#filtering)

### 使用的资源

`docker stats`

?> 默认情况下，`stats` 命令会每隔 1 秒钟刷新一次输出的内容直到你按下 `ctrl + c`。

如果不想持续的监控容器使用资源的情况，可以通过 `--no-stream` 选项只输出当前的状态：

`ocker stats --no-stream`

如果我们只想查看个别容器的资源使用情况，可以为 `docker stats` 命令显式的指定目标容器的名称或者是 ID：

`docker stats --no-stream <name1> <name2> ...`

## 占用空间

你可以通过以下命令来便捷的查看镜像、容器、数据卷所占用的空间。
```
$ docker system df

TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              24                  0                   1.992GB             1.992GB (100%)
Containers          1                   0                   62.82MB             62.82MB (100%)
Local Volumes       9                   0                   652.2MB             652.2MB (100%)
Build Cache                                                 0B                  0B
```



## 数据管理

### 数据卷

创建数据卷

```
docker volume create [volume-name]
```

查看所有数据卷

```
docker volume ls
```

查看数据卷的信息

```
docker volume inspect [volume-name]
```

挂载数据卷到容器

用 `--mount` 来挂载数据卷

例子：创建一个名为 `web` 的容器，并加载一个 `数据卷` 到容器的 `/webapp` 目录。

```
$ docker run -d -P \
    --name web \
    # -v my-vol:/wepapp \
    --mount source=my-vol,target=/webapp \
    training/webapp \
    python app.py
```

删除数据卷

```
docker volume rm [volume name]
```

!> `数据卷` 是被设计用来持久化数据的，它的生命周期独立于容器，Docker 不会在容器被删除后自动删除 `数据卷`，并且也不存在垃圾回收这样的机制来处理没有任何容器引用的 `数据卷`。如果需要在删除容器的同时移除数据卷。可以在删除容器的时候使用 `docker rm -v` 这个命令。

无主的数据卷可能会占据很多空间，要清理请使用以下命令

```
docker volume prune
```

### 监听主机目录

挂载一个主机目录作为数据卷

还是使用 `--mount`

```
$ docker run -d -P \
    --name web \
    # -v /src/webapp:/opt/webapp \
    --mount type=bind,source=/src/webapp,target=/opt/webapp \
    training/webapp \
    python app.py
```

!> 上面的命令加载主机的 `/src/webapp` 目录到容器的 `/opt/webapp` 目录。这个功能在进行测试的时候十分方便，比如用户可以放置一些程序到本地目录中，来查看容器是否正常工作。本地目录的路径必须是绝对路径。

Docker 挂载主机目录的默认权限是 `读写`，用户也可以通过增加 `readonly` 指定为 `只读`。

```
$ docker run -d -P \
    --name web \
    # -v /src/webapp:/opt/webapp:ro \
    --mount type=bind,source=/src/webapp,target=/opt/webapp,readonly \
    training/webapp \
    python app.py
```

`--mount` 标记也可以从主机挂载单个文件到容器中

```
$ docker run --rm -it \
   # -v $HOME/.bash_history:/root/.bash_history \
   --mount type=bind,source=$HOME/.bash_history,target=/root/.bash_history \
   ubuntu:17.10 \
   bash

root@2affd44b4667:/# history
1  ls
2  diskutil list
```

## 网络配置

### 端口映射

* `-P`: 随机映射一个 `49000~49900` 的端口到内部容器开放的网络端口。
* `-p`: 指定要映射的端口，并且，在一个指定端口上只可以绑定一个容器。支持的格式有 `ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort`。

!> 可以使用 `docker container ls` 查看端口绑定信息

可以通过 `docker container logs` 命令来查看容器内终端输出的信息。

```
docker container logs -f [ID or name]
```

`-f` 表示持续跟踪，不会立即返回。

!> `container` 可以省略不写：`docker logs -f [ID or name]`

映射所有接口地址

```
docker run -d -p 5000:5000 training/webapp python app.py
```

映射到指定地址的指定端口

```
docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py
```

映射到指定地址的任意端口

```
docker run -d -p 127.0.0.1::5000 training/webapp python app.py
```

`-p` 标记可以多次使用来绑定多个端口

```
$ docker run -d \
    -p 5000:5000 \
    -p 3000:80 \
    training/webapp \
    python app.py
```

### 容器互联

创建网络

```
docker network create -d bridge [net name]
```

`-d` 参数指定 Docker 网络类型，有 `bridge` 和 `overlay`。

将容器加入网络

```
docker run -it --rm --name centos1 --network [net name] centos sh
```

以此方式在启动一个容器 `centos2`

通过 ping 来证明 centos1 容器和 centos2 容器建立了互联关系。

在容器 `centos1` 中执行

```
ping centos2
```