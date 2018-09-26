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