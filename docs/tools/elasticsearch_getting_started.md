# 探索集群

## 群集运行状况

`GET /_cat/health?v`

举个栗子

``` bash
$ http GET http://localhost:9200/_cat/health\?v
HTTP/1.1 200 OK
content-encoding: gzip
content-length: 195
content-type: text/plain; charset=UTF-8

epoch      timestamp cluster        status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1538039367 09:09:27  docker-cluster yellow          1         1      6   6    0    0        5             0                  -                 54.5%
```

可以看到集群 `docker-cluster` 处于黄色状态。

状态一共有三种，绿色，黄色或红色。

* `Green` - 一切都很好（集群功能齐全）
* `Yellow` - 所有数据均可用，但尚未分配一些副本（群集功能齐全）
* `Red` - 某些数据由于某种原因不可用（群集部分功能）

我们还可以获得群集中的节点列表，如下所示：

```
GET /_cat/nodes?v
```

举个栗子


``` bash
$ http GET http://localhost:9200/_cat/nodes\?v 
HTTP/1.1 200 OK
content-encoding: gzip
content-length: 136
content-type: text/plain; charset=UTF-8

ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
172.20.0.2           45          86   7    1.06    1.06     1.09 mdi       *      DTanEEE
```

在这里，我们可以看到一个名为 “DTanEEE” 的节点，它是我们集群中当前的单个节点。

## 列出所有索引

```
GET /_cat/indices?v
```

举个栗子

``` bash
$ http GET http://localhost:9200/_cat/indices\?v
HTTP/1.1 200 OK
content-encoding: gzip
content-length: 207
content-type: text/plain; charset=UTF-8

health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .kibana  krdk8clyTvenh3zvRezK6A   1   0          1            0        4kb            4kb
yellow open   megacorp YJ4ZOjjvSKWmc3B9PwNmtA   5   1          3            0     17.2kb         17.2kb
```

## 创建索引

现在让我们创建一个名为 “customer” 的索引，然后再次列出所有索引：

```
PUT /customer?pretty
GET /_cat/indices?v
```

举个栗子

``` bash
$ http PUT http://localhost:9200/customer\?pretty
HTTP/1.1 200 OK
Warning: 299 Elasticsearch-6.4.0-595516e "the default number of shards will change from [5] to [1] in 7.0.0; if you wish to continue using the default of [5] shards, you must manage this on the create index request or with an index template" "Thu, 27 Sep 2018 09:45:12 GMT"
content-encoding: gzip
content-length: 83
content-type: application/json; charset=UTF-8

{
    "acknowledged": true,
    "index": "customer",
    "shards_acknowledged": true
}

$ http GET http://localhost:9200/_cat/indices\?v 
HTTP/1.1 200 OK
content-encoding: gzip
content-length: 252
content-type: text/plain; charset=UTF-8

health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .kibana  krdk8clyTvenh3zvRezK6A   1   0          1            0        4kb            4kb
yellow open   customer _6ix8CXITimJmfCtMHblvQ   5   1          0            0      1.1kb          1.1kb
yellow open   megacorp YJ4ZOjjvSKWmc3B9PwNmtA   5   1          3            0     17.2kb         17.2kb
```

我们现在有一个名为 customer 的索引，它有 5 个主分片和 1 个副本（默认值），并且它包含 0 个文档。

您可能还注意到客户索引标记了黄色运行状况。回想一下我们之前的讨论，黄色表示某些副本尚未分配。此索引发生这种情况的原因是因为默认情况下 Elasticsearch 为此索引创建了一个副本。由于我们目前只有一个节点在运行，因此在另一个节点加入集群的较晚时间点之前，尚无法分配一个副本（用于高可用性）。将该副本分配到第二个节点后，此索引的运行状况将变为绿色。

## 索引和创建文档

现在让我们在 customer 索引中加入一些内容。我们将一个简单的 customer 文档索引到 customer 索引中，ID 为 1，如下所示：

``` bash
PUT /customer/_doc/1?pretty
{
  "name": "John Doe"
}
```

举个栗子

``` bash
$ http PUT http://localhost:9200/customer/_doc/1\?pretty name="John Doe"
HTTP/1.1 201 Created
Location: /customer/_doc/1
content-encoding: gzip
content-length: 161
content-type: application/json; charset=UTF-8

{
    "_id": "1",
    "_index": "customer",
    "_primary_term": 2,
    "_seq_no": 0,
    "_shards": {
        "failed": 0,
        "successful": 1,
        "total": 2
    },
    "_type": "_doc",
    "_version": 1,
    "result": "created"
}
```

从上面可以看出，在 customer 索引中成功创建了一个新的 customer 文档。该文档还具有我们在索引时指定的内部标识 1。

值得注意的是，Elasticsearch 在您将文档编入索引之前不需要先显式创建索引。在前面的示例中，如果 customer 索引事先尚未存在，则 Elasticsearch 将自动创建 customer 索引。

现在让我们检索刚刚编入索引的文档：

``` bash
GET /customer/_doc/1?pretty
```

举个栗子

``` bash
$ http GET http://localhost:9200/customer/_doc/1\?pretty              
HTTP/1.1 200 OK
content-encoding: gzip
content-length: 125
content-type: application/json; charset=UTF-8

{
    "_id": "1",
    "_index": "customer",
    "_source": {
        "name": "John Doe"
    },
    "_type": "_doc",
    "_version": 1,
    "found": true
}
```

除了字段之外，没有任何异常，发现，说明我们找到了一个具有请求 ID `1` 的文档和另一个字段 `_source` ，它返回我们从上一步索引的完整 JSON 文档。

## 删除索引

现在让我们删除刚刚创建的索引，然后再次列出所有索引：

``` bash
DELETE /customer?pretty
GET /_cat/indices?v
```

举个栗子

``` bash
$ http DELETE http://localhost:9200/customer\?pretty       
HTTP/1.1 200 OK
content-encoding: gzip
content-length: 54
content-type: application/json; charset=UTF-8

{
    "acknowledged": true
}

$ http GET http://localhost:9200/_cat/indices\?v        
HTTP/1.1 200 OK
content-encoding: gzip
content-length: 207
content-type: text/plain; charset=UTF-8

health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .kibana  krdk8clyTvenh3zvRezK6A   1   0          1            0        4kb            4kb
yellow open   megacorp YJ4ZOjjvSKWmc3B9PwNmtA   5   1          3            0     17.2kb         17.2kb
```

这意味着索引已成功删除

在我们继续之前，让我们再仔细看看到目前为止我们学到的一些 API 命令：

``` bash
PUT /customer
PUT /customer/_doc/1
{
  "name": "John Doe"
}
GET /customer/_doc/1
DELETE /customer
```

如果我们仔细研究上述命令，我们实际上可以看到我们如何在 Elasticsearch 中访问数据的模式。该模式可归纳如下：

``` bash
<REST Verb> /<Index>/<Type>/<ID>
```

这种 REST 访问模式在所有 API 命令中都非常普遍，如果您能够简单地记住它，您将在掌握 Elasticsearch 方面有一个良好的开端。

# 修改你的数据

Elasticsearch 几乎实时提供数据操作和搜索功能。默认情况下，从索引/更新/删除数据到搜索结果中显示的时间，您可能会有一秒钟的延迟（刷新间隔）。这是与 SQL 等其他平台的重要区别，其中数据在事务完成后立即可用。

## 索引替换文档

我们之前已经看到了如何索引单个文档。让我们再次回想一下这个命令：

``` bash
PUT /customer/_doc/1?pretty
{
  "name": "John Doe"
}
```

同样，上面将指定的文档索引到 customer 索引中，ID 为 1。如果我们再使用不同（或相同）的文档再次执行上述命令，Elasticsearch 将在 ID 为 1 的现有文档之上替换（即重新索引）新文档：

``` bash
PUT /customer/_doc/1?pretty
{
  "name": "Jane Doe"
}
```

以上内容将 ID 为 `1` 的文档名称从 “John Doe” 更改为 “Jane Doe”。另一方面，如果我们使用不同的 ID，则会对新文档编制索引，并且索引中已有的现有文档保持不变。

``` bash
PUT /customer/_doc/2?pretty
{
  "name": "Jane Doe"
}
```

以上索引 ID 为 2 的新文档。

索引时，ID 部分是可选的。如果未指定，Elasticsearch 将生成随机 ID，然后使用它来索引文档。Elasticsearch 生成的实际 ID（或前面示例中显式指定的内容）将作为索引 API 调用的一部分返回。

此示例显示如何在没有显式 ID 的情况下索引文档：

``` bash
POST /customer/_doc?pretty
{
  "name": "Jane Doe"
}
```

!> 请注意，在上述情况下，我们使用 `POST` 动词而不是 `PUT` ，因为我们没有指定 `ID` 。

## 更新文档

除了能够索引和替换文档，我们还可以更新文档。请注意，Elasticsearch 实际上并没有在内部进行就地更新。每当我们进行更新时，Elasticsearch 都会删除旧文档，然后一次性对应用了更新的新文档编制索引。

此示例显示如何通过将名称字段更改为 “Jane Doe” 来更新以前的文档（ID 为 1）：

``` bash
POST /customer/_doc/1/_update?pretty
{
  "doc": { "name": "Jane Doe" }
}
```

此示例显示如何通过将名称字段更改为 “Jane Doe” 来更新我们以前的文档（ID 为 1），同时向其添加年龄字段：

``` bash
POST /customer/_doc/1/_update?pretty
{
  "doc": { "name": "Jane Doe", "age": 20 }
}
```

也可以使用简单脚本执行更新。此示例使用脚本将年龄增加 5：

``` bash
POST /customer/_doc/1/_update?pretty
{
  "script" : "ctx._source.age += 5"
}
```

在上面的示例中，`ctx._source` 指的是即将更新的当前源文档。

Elasticsearch 提供了在给定查询条件（如 SQL UPDATE-WHERE 语句）的情况下更新多个文档的功能。请参阅 [docs-update-by-query API](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/docs-update-by-query.html)

## 删除文档

删除文档非常简单。此示例显示如何删除 ID 为 `2` 的以前的客户：

``` bash
DELETE /customer/_doc/2?pretty
```

请参阅 [_delete_by_query API](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/docs-delete-by-query.html) 以删除与特定查询匹配的所有文档。值得注意的是，删除整个索引而不是使用 Delete By Query API 删除所有文档会更有效。

## 批量处理

除了能够索引，更新和删除单个文档之外，Elasticsearch 还提供了使用 [_bulk API](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/docs-bulk.html) 批量执行上述任何操作的功能。此功能非常重要，因为它提供了一种非常有效的机制，可以尽可能快地执行多个操作，并尽可能少地进行网络往返。

作为一个简单示例，以下调用在一个批量操作中索引两个文档（ID 1 - John Doe 和 ID 2 - Jane Doe）：

``` bash
POST /customer/_doc/_bulk?pretty
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }
```

此示例更新第一个文档（ID 为 1），然后在一个批量操作中删除第二个文档（ID 为 2）：

``` bash
POST /customer/_doc/_bulk?pretty
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}
```

请注意，对于删除操作，之后没有相应的源文档，因为删除只需要删除文档的 ID。

Bulk API 不会因其中一个操作失败而失败。如果单个操作因任何原因失败，它将继续处理其后的其余操作。批量 API 返回时，它将为每个操作提供一个状态（按照发送的顺序），以便您可以检查特定操作是否失败。

# 探索你的数据

样本数据集

现在我们已经了解了基础知识，让我们尝试更真实的数据集。我准备了一份关于客户银行账户信息的虚构 JSON 文档样本。每个文档都有以下架构：

``` json
{
    "account_number": 0,
    "balance": 16623,
    "firstname": "Bradshaw",
    "lastname": "Mckenzie",
    "age": 29,
    "gender": "F",
    "address": "244 Columbus Place",
    "employer": "Euron",
    "email": "bradshawmckenzie@euron.com",
    "city": "Hobucken",
    "state": "CO"
}
```

奇怪的是，这些数据是使用 [www.json-generator.com/](http://www.json-generator.com/) 生成的，因此请忽略数据的实际值和语义，因为这些都是随机生成的。

加载样本数据

您可以从[此处](https://github.com/elastic/elasticsearch/blob/master/docs/src/test/resources/accounts.json?raw=true)下载示例数据集（accounts.json）。将它解压缩到我们当前的目录，然后将它们加载到我们的集群中，如下所示：

``` bash
# 下载样本数据
$ http https://raw.githubusercontent.com/elastic/elasticsearch/master/docs/src/test/resources/accounts.json > accounts.json
# 请求 es
$ http POST http://localhost:9200/bank/_doc/_bulk\?pretty\&refresh < accounts.json
```

查看索引

``` bash
$ http GET http://localhost:9200/_cat/indices\?v    
HTTP/1.1 200 OK
content-encoding: gzip
content-length: 204
content-type: text/plain; charset=UTF-8

health status index   uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .kibana krdk8clyTvenh3zvRezK6A   1   0          1            0        4kb            4kb
yellow open   bank    g0pKm9uHRGyDKo06smITIA   5   1       1000            0    475.1kb        475.1kb
```

这意味着我们只是成功地将 1000 个文档批量索引到 bank 索引（在 _doc 类型下）。

## 搜索 API

现在让我们从一些简单的搜索开始吧。运行搜索有两种基本方法：一种是通过 [REST request URI](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/search-uri-request.html) 发送搜索参数，另一种是通过 [REST request body](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/search-request-body.html) 发送搜索参数。请求体方法允许您更具表现力，并以更易读的 JSON 格式定义搜索。我们将尝试一个请求 URI 方法的示例，但是对于本教程的其余部分，我们将专门使用请求体方法。

可以从 `_search` 端点访问用于搜索的 REST API。此示例返回 bank 索引中的所有文档：

``` bash
GET /bank/_search?q=*&sort=account_number:asc&pretty
```

让我们首先剖析搜索调用。我们在 bank 索引中搜索（_search endpoint），`q=*` 参数指示 Elasticsearch 匹配索引中的所有文档。`sort=account_number:asc` 参数指示使用升序中的每个文档的 `account_number` 字段对结果进行排序。`pretty` 参数再次告诉 Elasticsearch 返回漂亮的 JSON 结果。

响应（部分显示）：

``` json
{
  "took" : 63,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : null,
    "hits" : [ {
      "_index" : "bank",
      "_type" : "_doc",
      "_id" : "0",
      "sort": [0],
      "_score" : null,
      "_source" : {"account_number":0,"balance":16623,"firstname":"Bradshaw","lastname":"Mckenzie","age":29,"gender":"F","address":"244 Columbus Place","employer":"Euron","email":"bradshawmckenzie@euron.com","city":"Hobucken","state":"CO"}
    }, {
      "_index" : "bank",
      "_type" : "_doc",
      "_id" : "1",
      "sort": [1],
      "_score" : null,
      "_source" : {"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
    }, ...
    ]
  }
}
```

至于响应，我们看到以下部分：

* `took` - Elasticsearch 执行搜索的时间（以毫秒为单位）
* `timed_out` – 告诉我们搜索是否超时
* `_shards` – 告诉我们搜索了多少个分片，以及搜索成功/失败分片的计数
* `hits` – 搜索结果
* `hits.total` – 符合我们搜索条件的文件总数
* `hits.hits` – 实际的搜索结果数组（默认为前 10 个文档）
* `hits.sort` - 对结果进行排序（如果按分数排序则没有此字段）
* `hits._score` 和 `max_score` - 暂时忽略这些字段

以下是使用请求正文方法替代的上述完全相同的搜索：

``` bash
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
```

这里的区别在于，我们不是在 URI 中传递 `q=*`，而是向 `_search API` 提供 JSON 样式的查询请求体。我们将在下一节讨论这个 JSON 查询。

重要的是要理解，一旦您获得了搜索结果，Elasticsearch 就完全完成了请求，并且不会在结果中维护任何类型的服务器端资源或打开游标。这与 SQL 等许多其他平台形成鲜明对比，其中您可能最初预先获得查询结果的部分子集，然后如果要获取（或翻页）其余的则必须连续返回服务器 使用某种有状态服务器端游标的结果。

## 介绍查询语言

Elasticsearch 提供了一种 JSON 样式的领域特定语言，可用于执行查询。这被称为 [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/query-dsl.html)。查询语言非常全面，乍一看可能令人生畏，但实际学习它的最佳方法是从一些基本示例开始。

回到上一个例子，我们执行了这个查询：

``` bash
GET /bank/_search
{
  "query": { "match_all": {} }
}
```

解析上面的内容， `query` 部分告诉我们查询定义是什么， `match_all` 部分只是我们想要运行的查询类型。 `match_all` 查询只是搜索指定索引中的所有文档。

除了查询参数，我们还可以传递其他参数来影响搜索结果。在上面的部分示例中，我们传递了 `sort` ，这里我们传入大小：

``` bash
GET /bank/_search
{
  "query": { "match_all": {} },
  "size": 1
}
```

请注意，如果未指定大小，则默认为 10。

此示例执行 `match_all` 并返回文档 10 到 19：

``` bash
GET /bank/_search
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}
```

`from` 参数（从 `0` 开始）指定从哪个文档索引开始， `size` 参数指定从 `from` 参数开始返回的文档数。在实现搜索结果的分页时，此功能非常有用。请注意，如果未指定 `from` ，则默认为 `0`。

此示例执行 `match_all` 并按帐户余额降序对结果进行排序，并返回前 `10` 个（默认大小）文档。

``` bash
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": { "balance": { "order": "desc" } }
}
```

## 搜索

现在我们已经看到了一些基本的搜索参数，让我们再深入研究一下 Query DSL。我们先来看一下返回的文档字段。默认情况下，完整的 JSON 文档作为所有搜索的一部分返回。这被称为源（搜索命中中的 `_source` 字段）。如果我们不希望返回整个源文档，我们只能请求返回源中的几个字段。

此示例显示如何从搜索中返回两个字段， `account_number` 和 `balance` （在 `_source` 内部）：

``` bash
GET /bank/_search
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"]
}
```

请注意，上面的示例只是简化了 `_source` 字段。`_source` 字段内部只包含字段 `account_number` 和 `balance` 。

如果您学过 SQL ，则上述内容在概念上与 SQL SELECT FROM 字段列表有些相似。

现在让我们转到查询部分。以前，我们已经看到 `match_all` 查询如何用于匹配所有文档。现在让我们介绍一个名为 [match query](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/query-dsl-match-query.html) 的新查询，它可以被认为是一个基本的字段搜索查询（即针对特定字段或字段集进行的搜索）。

此示例返回编号为 20 的帐户：

``` bash
GET /bank/_search
{
  "query": { "match": { "account_number": 20 } }
}
```

此示例返回地址中包含术语 “mill” 的所有帐户：

``` bash
GET /bank/_search
{
  "query": { "match": { "address": "mill" } }
}
```

此示例返回地址中包含术语 “mill” 或 “lane” 的所有帐户：

``` bash
GET /bank/_search
{
  "query": { "match": { "address": "mill lane" } }
}
```

此示例是 match（match_phrase）的变体，它返回地址中包含短语 “mill lane” 的所有帐户：

``` bash
GET /bank/_search
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
```

我们现在介绍 [bool query](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/query-dsl-bool-query.html)。 `bool` 查询允许我们使用布尔逻辑将较小的查询组成更大的查询。

此示例组成两个匹配查询，并返回地址中包含 “mill” 和 “lane” 的所有帐户：

``` bash
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

在上面的示例中，`bool must` 子句指定所有必须为 `true` 的查询才能将文档视为匹配项。

相反，此示例组成两个匹配查询并返回地址中包含 “mill” 或 “lane” 的所有帐户：

``` bash
GET /bank/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

在上面的示例中，`bool should` 子句指定了一个查询列表，其中任何一个查询为 `true` 就能将文档视为匹配项。

此示例组成两个匹配查询，并返回地址中既不包含 “mill” 也不包含 “lane” 的所有帐户：

``` bash
GET /bank/_search
{
  "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

在上面的示例中，`bool must_not` 子句指定了一个查询列表，对于文档而言，这些查询都不能为 `true`。

我们可以在 `bool` 查询中同时组合 `must` ， `should` 和 `must_not` 子句。此外，我们可以在任何这些 `bool` 子句中组合 `bool` 查询来模仿任何复杂的多级布尔逻辑。

此示例返回任何 40 岁但不住在 ID（aho）的人的所有帐户：

``` bash
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
```

## 过滤

在上一节中，我们跳过了一个称为文档分数的小细节（搜索结果中的 `_score` 字段）。分数是一个数值，它是文档与我们指定的搜索查询匹配程度的相对度量。分数越高，文档越相关，分数越低，文档的相关性越低。

但是查询并不总是需要产生分数，特别是当它们仅用于 “过滤” 文档集时。Elasticsearch 会检测这些情况并自动优化查询执行，以便不计算无用的分数。

我们在上一节中介绍的 `bool` 查询还支持过滤子句，这些子句允许使用查询来限制将与其他子句匹配的文档，而不会更改计算得分的方式。作为一个例子，让我们介绍 [range query](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/query-dsl-range-query.html) ，它允许我们按一系列值过滤文档。这通常用于数字或日期过滤。

此示例使用 `bool` 查询返回所有余额介于 20000 和 30000 之间的帐户。换句话说，我们希望找到余额大于或等于 20000 且小于或等于 30000 的帐户。

``` bash
GET /bank/_search
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
```

解析上面的内容， `bool` 查询包含 `match_all` 查询（查询部分）和范围查询（过滤器部分）。我们可以将任何其他查询替换为查询和过滤器部分。在上述情况下，范围查询非常有意义，因为落入范围的文档都 “相同” 匹配，即，没有文档比另一文档更相关。

除了 `match_all` ， `match` ， `bool` 和 `range` 查询之外，还有很多其他可用的查询类型，我们不会在这里讨论它们。由于我们已经基本了解它们的工作原理，因此将这些知识应用于学习和试验其他查询类型应该不会太困难。

## 聚合

聚合提供了从数据中分组和提取统计信息的功能。考虑聚合的最简单方法是将其大致等同于 SQL GROUP BY 和 SQL 聚合函数。在 Elasticsearch 中，您可以执行返回匹配的搜索，同时在一个响应中返回与命中相关的聚合结果。这是非常强大和高效的，因为您可以运行查询和多个聚合，并一次性获取两个（或任一）操作的结果，避免使用简洁和简化的 API 进行网络往返。

首先，此示例按状态对所有帐户进行分组，然后返回按计数降序排序的前 10 个（默认）状态（也是默认值）：

``` bash
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
```

在 SQL 中，上述聚合在概念上类似于：

``` sql
SELECT state, COUNT(*) FROM bank GROUP BY state ORDER BY COUNT(*) DESC LIMIT 10;
```

响应（部分显示）：

``` bash
{
  "took": 29,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped" : 0,
    "failed": 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "doc_count_error_upper_bound": 20,
      "sum_other_doc_count": 770,
      "buckets" : [ {
        "key" : "ID",
        "doc_count" : 27
      }, {
        "key" : "TX",
        "doc_count" : 27
      }, {
        "key" : "AL",
        "doc_count" : 25
      }, {
        "key" : "MD",
        "doc_count" : 25
      }, {
        "key" : "TN",
        "doc_count" : 23
      }, {
        "key" : "MA",
        "doc_count" : 21
      }, {
        "key" : "NC",
        "doc_count" : 21
      }, {
        "key" : "ND",
        "doc_count" : 21
      }, {
        "key" : "ME",
        "doc_count" : 20
      }, {
        "key" : "MO",
        "doc_count" : 20
      } ]
    }
  }
}
```

我们可以看到 ID（爱达荷州）有27个账户，其次是 TX（德克萨斯州）的 27 个账户，其次是 AL（阿拉巴马州）的 25 个账户，依此类推。

请注意，我们将 `size=0` 设置为不显示搜索命中，因为我们只想在响应中看到聚合结果。

在前一个聚合的基础上，此示例按州计算平均帐户余额（同样仅针对按降序排序的前 10 个州）：

``` bash
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

请注意我们如何嵌套 `group_by_state` 聚合中的 `average_balance` 聚合。这是所有聚合的常见模式。您可以在聚合中任意嵌套聚合，以从数据中提取所需的轮转摘要。

在前一个聚合的基础上，我们现在按降序排列平均余额：

``` bash
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

此示例演示了我们如何按年龄段（20-29岁，30-39岁和40-49岁）进行分组，然后按性别分组，最后得到每个年龄段的平均帐户余额：

``` bash
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_age": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "from": 20,
            "to": 30
          },
          {
            "from": 30,
            "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      },
      "aggs": {
        "group_by_gender": {
          "terms": {
            "field": "gender.keyword"
          },
          "aggs": {
            "average_balance": {
              "avg": {
                "field": "balance"
              }
            }
          }
        }
      }
    }
  }
}
```

还有许多其他聚合功能，我们在此不再详述。如果您想进行进一步的实验，[聚合参考指南](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/search-aggregations.html)是一个很好的起点。