# redis

https://hub.docker.com/_/redis/

下载

``` bash
docker pull redis
```

启动

``` bash
docker run --name test-redis -p 6379:6379 -d redis
```

# mongo

下载

``` bash
docker pull mongo
```

启动

没密码方式

``` bash
docker run -d --name test-mongo -p 27017:27017  mongo
```

有密码方式

``` bash
docker run -d --name some-mongo -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=mongoadmin -e MONGO_INITDB_ROOT_PASSWORD=123456 mongo
```

# mysql

https://hub.docker.com/_/mysql/

下载

``` bash
docker pull mysql
```

启动

默认是最新的 mysql 版本

``` bash
docker run --name test-mysql -e MYSQL_ROOT_PASSWORD=123456 -p 3307:3306 -d mysql
```

指定 mysql 版本

``` bash
docker run --name test-mysql -e MYSQL_ROOT_PASSWORD=123456 -p 3307:3306 -d mysql:5
```

指定编码

```
docker run --name test-mysql -e MYSQL_ROOT_PASSWORD=123456 -p 3307:3306 -d mysql:5 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

查看所有可用配置

```
docker run -it --rm mysql:tag --verbose --help
```

访问

``` bash
mysql -h 127.0.0.1 -P 3307 -u root -p
```

# zookeeper

https://hub.docker.com/_/zookeeper/

下载

``` bash
docker pull zookeeper
```

启动

``` bash
docker run --name test-zookeeper -p 2181:2181 -d zookeeper
```

默认会有三个端口 `EXPOSE 2181 2888 3888`,（ zookeeper 客户端端口，跟随端口，选举端口）

# Elasticsearch

https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html

下载

``` bash
docker pull docker.elastic.co/elasticsearch/elasticsearch:6.4.0
```

启动

``` bash
docker run --name test-es -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -d docker.elastic.co/elasticsearch/elasticsearch:6.4.0
```

检查

``` bash
$ http GET http://127.0.0.1:9200/_cat/health
HTTP/1.1 200 OK
content-encoding: gzip
content-length: 81
content-type: text/plain; charset=UTF-8

1537965077 12:31:17 docker-cluster green 1 1 0 0 0 0 0 0 - 100.0%
```

或者

``` bash
$ http GET http://localhost:9200/\?pretty                                 
HTTP/1.1 200 OK
content-encoding: gzip
content-length: 298
content-type: application/json; charset=UTF-8

{
    "cluster_name": "docker-cluster",
    "cluster_uuid": "GfeW_4xTROWf1c1qeUkhPw",
    "name": "JTcs_Nq",
    "tagline": "You Know, for Search",
    "version": {
        "build_date": "2018-08-17T23:18:47.308994Z",
        "build_flavor": "default",
        "build_hash": "595516e",
        "build_snapshot": false,
        "build_type": "tar",
        "lucene_version": "7.4.0",
        "minimum_index_compatibility_version": "5.0.0",
        "minimum_wire_compatibility_version": "5.6.0",
        "number": "6.4.0"
    }
}
```

# Kibana

https://www.elastic.co/guide/en/kibana/current/docker.html
https://www.elastic.co/guide/cn/kibana/current/docker.html

下载

``` bash
docker pull docker.elastic.co/kibana/kibana:6.4.1
```

配合 Elasticsearch 使用：

首先创建网络

``` bash
docker network create -d bridge es-net
```

在启动 Elasticsearch 容器和 Kibana 容器时加入网络配置

``` bash
docker run --name test-es -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" --network es-net -d docker.elastic.co/elasticsearch/elasticsearch:6.4.0

docker run --name test-kibana -p 5601:5601 -e "ELASTICSEARCH_URL=http://test-es:9200/" --network es-net -d docker.elastic.co/kibana/kibana:6.4.1
```

默认环境变量

```
server.name=kibana
server.host="0"
elasticsearch.url=http://elasticsearch:9200
xpack.monitoring.ui.container.elasticsearch.enabled=true
```

在 docker 命令中配置环境变量时，所有参数均以大写加下划线表示

```
SERVER_NAME <=> server.name
KIBANA_DEFAULTAPPID <=> kibana.defaultAppId
XPACK_MONITORING_ENABLED <=> xpack.monitoring.enabled
```

更多配置项见

https://www.elastic.co/guide/en/kibana/current/settings.html

https://www.elastic.co/guide/en/kibana/current/settings-xpack-kb.html

# Rabbit MQ

https://hub.docker.com/_/rabbitmq/

```
docker pull rabbitmq
```

启动，运行守护进程

关于RabbitMQ的一个重要注意事项是它根据所谓的“节点名称”存储数据，默认为主机名。这对于在 Docker 中的使用意味着我们应该为每个守护进程明确指定 `-h / -hostname`，这样我们就不会获得随机主机名并且可以跟踪我们的数据：

```
docker run -d --hostname my-rabbit --name some-rabbit -p 5672:5672 rabbitmq:3
```

这将启动一个侦听默认端口 `5672` 的 RabbitMQ 容器。如果你给它一分钟，然后做 `docker logs some-rabbit`，你会在输出中看到一个类似于的块：

```
 node           : rabbit@my-rabbit
 home dir       : /var/lib/rabbitmq
 config file(s) : /etc/rabbitmq/rabbitmq.conf
 cookie hash    : RCxPSNwPOaHSsvGe2TKjig==
 log(s)         : <stdout>
 database dir   : /var/lib/rabbitmq/mnesia/rabbit@my-rabbit
```

如果您希望更改 `guest/guest` 的默认用户名和密码，可以使用 `RABBITMQ_DEFAULT_USER` 和 `RABBITMQ_DEFAULT_PASS` 环境变量：

``` bash
$ docker run -d --hostname my-rabbit --name some-rabbit -e RABBITMQ_DEFAULT_USER=user -e RABBITMQ_DEFAULT_PASS=password rabbitmq:3-management
```

然后，您可以在浏览器中访问 `http://localhost:8080` 或 `http://host-ip:8080` 并使用 `user/password` 访问管理控制台

# HBase

https://hub.docker.com/r/harisekhon/hbase

``` bash
git clone https://github.com/HariSekhon/Dockerfiles.git
cd Dockerfiles\hbase
docker-compose up
```

管理地址

* http://localhost:16010/master-status for the Master Server
* http://localhost:9095/thrift.jsp for the thrift UI
* http://localhost:8085/rest.jsp for the REST server UI
* http://localhost:16010/zk.jsp for the embedded Zookeeper

进入 hbase shell

``` bash
docker exec -it [container id] /hbase/bin/hbase shell
```

python 客户端测试

``` python
>>> import happybase
>>> connection = happybase.Connection('127.0.0.1', 9090)
>>> connection.create_table('table-name', { 'family': dict() } )
>>> connection.tables()
['table-name']
>>> table = connection.table('table-name')
>>> table.put('row-key', {'family:qual1': 'value1', 'family:qual2': 'value2'})
>>> for k, data in table.scan():
...   print(k, data)
...
row-key {'family:qual1': 'value1', 'family:qual2': 'value2'}
>>>
```