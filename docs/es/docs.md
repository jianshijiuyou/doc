# index API

索引 API 在特定索引中添加或更新类型化的 JSON 文档，使其可搜索。以下示例将 JSON 文档插入到 “twitter” 索引中，类型为 “_doc” 且 ID 为 1 的类型下：

``` json
PUT twitter/_doc/1
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

上述索引操作的结果是：

``` json
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 1,
    "_seq_no" : 0,
    "_primary_term" : 1,
    "result" : "created"
}
```

`_shards` 标头提供有关索引操作的复制过程的信息。

* `total` - 指示应在其上执行索引操作的分片副本（主分片和副本分片）的数量。
* `successful` - 表示索引操作成功的分片副本数。
* `failed` - 在副本分片上索引操作失败的情况下包含复制相关错误的数组。

在成功至少为 1 的情况下，索引操作成功。

?> 索引操作成功返回时，可能无法全部启动副本分片（默认情况下，只需要主分区，但可以更改此行为）。在这种情况下，`total` 将等于基于 `number_of_replicas` 设置的总分片数，并且 `success` 将等于启动的分片数（primary plus replicas）。如果没有失败，则 `failed` 将为 0。

## 自动索引创建

如果之前尚未创建索引（索引用于手动创建索引的 create index API），则索引操作会自动创建索引，如果尚未创建索引，则还会自动为特定类型创建动态类型映射（签出 put mapping API 用于手动创建类型映射）。

映射本身非常灵活，无需架构。新字段和对象将自动添加到指定类型的映射定义中。

通过在所有节点的配置文件中将 action.auto_create_index 设置为 false，可以禁用自动索引创建。通过将 index.mapper.dynamic 设置为 false per-index 作为索引设置，可以禁用自动映射创建。

自动索引创建可以包括基于模式的白/黑列表，例如，set action.auto_create_index to + aaa *， - bbb *，+ ccc *， - *（+表示允许， - 表示不允许）。

## 版本

每个索引文档都有一个版本号。关联的版本号作为对索引 API 请求的响应的一部分返回。索引 API 可选地允许在指定 version参 数时进行乐观并发控制。这将控制要针对该操作执行的文档的版本。用于版本控制的用例的一个很好的例子是执行事务性读取然后更新。从最初读取的文档中指定版本可确保在此期间未发生任何更改（在读取时为了更新，建议将首选项设置为 _primary ）。例如：

``` json
PUT twitter/_doc/1?version=2
{
    "message" : "elasticsearch now has versioning support, double cool!"
}
```

注意：版本控制是完全实时的，并且不受搜索操作的近实时方面的影响。如果未提供版本，则执行该操作而不进行任何版本检查。

默认情况下，使用从1开始的内部版本控制，并随每次更新而增加，包括删除。可选地，版本号可以用外部值补充（例如，如果在数据库中维护）。要启用此功能，version_type应将其设置为 external。提供的值必须是数字，长值大于或等于0，小于约9.2e + 18。使用外部版本类型时，系统会检查传递给索引请求的版本号是否大于当前存储文档的版本，而不是检查匹配的版本号。如果为true，则将索引文档并使用新版本号。如果提供的值小于或等于存储文档的版本号，则会发生版本冲突，索引操作将失败。

## 版本类型

在上面解释的 internal＆externalversion 类型旁边，Elasticsearch 还支持特定用例的其他类型。以下是不同版本类型及其语义的概述。

* internal  
    如果给定版本与存储文档的版本相同，则仅索引文档。
* external or external_gt  
    如果给定版本严格高于存储文档的版本或者没有现有文档，则仅索引文档。给定版本将用作新版本，并将与新文档一起存储。提供的版本必须是非负长号。
* external_gte  
    仅在给定版本等于或高于存储文档的版本时索引文档。如果没有现有文档，操作也将成功。给定版本将用作新版本，并将与新文档一起存储。提供的版本必须是非负长号。

注意：external_gte版本类型适用于特殊用例，应小心使用。如果使用不当，可能会导致数据丢失。还有另一个选项，force它已被弃用，因为它可能导致主分片和副本分片发散。

## 操作类型

索引操作还接受op_type可用于强制create操作的操作，允许“put-if-absent”行为。当 create使用时，如果该ID在文档中的索引已经存在索引操作将失败。

以下是使用 op_type 参数的示例：

``` json
PUT twitter/_doc/1?op_type=create
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

指定 create 的另一个选项是使用以下 uri：

``` json
PUT twitter/_doc/1/_create
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

## 自动 ID 生成

可以在不指定 id 的情况下执行索引操作。在这种情况下，将自动生成id。此外， `op_type` 将自动设置为 `create` 。这是一个例子（注意使用 POST 而不是 PUT ）：

``` json
POST twitter/_doc/
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

上述索引操作的结果是：

``` json
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "W0tpsmIBdwcYyG50zbta",
    "_version" : 1,
    "_seq_no" : 0,
    "_primary_term" : 1,
    "result": "created"
}
```

## 路由

默认情况下，通过使用文档的 id 值的哈希来控制分片放置 - 或路由。对于更明确的控制，可以使用路由参数在每个操作的基础上直接指定输入到路由器使用的散列函数的值。例如：

``` json
POST twitter/_doc?routing=kimchy
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

在上面的示例中，“_ doc” 文档根据提供的路由参数路由到分片：“kimchy”。

在设置显式映射时，可以选择使用 `_routing` 字段来指示索引操作从文档本身中提取路由值。这确实来自另一个文档解析过程的（非常小的）成本。如果定义了 `_routing` 映射并将其设置为必需，则如果未提供或提取路由值，则索引操作将失败。

## 分布式

索引操作根据其路由指向主分片（请参阅上面的“路由”部分），并在包含此分片的实际节点上执行。主分片完成操作后，如果需要，更新将分发到适用的副本。

## 等待活动碎片

为了提高对系统写入的弹性，可以将索引操作配置为在继续操作之前等待一定数量的活动分片副本。如果必需数量的活动分片副本不可用，则写入操作必须等待并重试，直到必需的分片副本已启动或发生超时。默认情况下，写入操作仅等待主分片在继续（即wait_for_active_shards=1）之前处于活动状态。可以通过设置动态地在索引设置中覆盖此默认值index.write.wait_for_active_shards。要更改每个操作的此行为，wait_for_active_shards可以使用request参数。

有效值是 all 或任何正整数，直到索引中每个分片的已配置副本总数（即 number_of_replicas+1）。指定负值或大于分片副本数的数字将引发错误。

## 超时

执行索引操作时，分配用于执行索引操作的主分片可能不可用。造成这种情况的一些原因可能是主分片当前正在从网关恢复或正在进行重定位。默认情况下，索引操作将在主分片上等待最多1分钟，然后失败并响应错误。 `timeout` 参数可用于显式指定等待的时间。以下是将其设置为 5 分钟的示例：

``` json
PUT twitter/_doc/1?timeout=5m
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

# Get API

get API 允许根据其 id 从索引中获取类型化的 JSON 文档。以下示例从名为 `twitter` 的索引获取 JSON 文档，该索引名为 `_doc` ，id 值为 `0` ：

``` json
GET twitter/_doc/0
```

上述 get 操作的结果是：

``` json
{
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "0",
    "_version" : 1,
    "found": true,
    "_source" : {
        "user" : "kimchy",
        "date" : "2009-11-15T14:12:12",
        "likes": 0,
        "message" : "trying out Elasticsearch"
    }
}
```

上面的结果包括我们希望检索的文档的 `_index` ， `_type` ， `_id` 和 `_version` ，包括文档的实际 `_source` （如果可以找到）（如响应中的 `found` 字段所示）。

API 还允许使用 HEAD 检查文档是否存在，例如：

``` json
HEAD twitter/_doc/0
```

## 实时

默认情况下，get API 是实时的，并且不受索引刷新率的影响（当数据对搜索可见时）。如果文档已更新但尚未刷新，则get API将就地发出刷新调用以使文档可见。这也将使上次刷新后其他文档发生变化。为了禁用实时GET，可以将realtime参数设置为false。

## Source 过滤

默认情况下，除非已使用 `stored_fields` 参数或禁用了 `_source` 字段，否则 get 操作将返回 `_source` 字段的内容。您可以使用 `_source` 参数关闭 `_source` 检索：

``` json
GET twitter/_doc/0?_source=false
```

如果您只需要完整 `_source` 中的一个或两个字段，则可以使用 `_source_include` 和 `_source_exclude` 参数来包含或过滤掉您需要的部分。这对于大型文档尤其有用，其中部分检索可以节省网络开销。这两个参数都使用逗号分隔的字段列表或通配符表达式。例：

``` json
GET twitter/_doc/0?_source_include=*.id&_source_exclude=entities
```

如果您只想指定包含，则可以使用较短的表示法：

``` json
GET twitter/_doc/0?_source=*.id,retweeted
```

## 存储的字段

get 操作允许指定将通过传递 `stored_fields` 参数返回的一组存储字段。如果未存储请求的字段，则将忽略它们。例如，考虑以下映射：

``` json
PUT twitter
{
   "mappings": {
      "_doc": {
         "properties": {
            "counter": {
               "type": "integer",
               "store": false
            },
            "tags": {
               "type": "keyword",
               "store": true
            }
         }
      }
   }
}
```

现在我们可以添加一个文档：

``` json
PUT twitter/_doc/1
{
    "counter" : 1,
    "tags" : ["red"]
}
```

并尝试检索它：

``` json
GET twitter/_doc/1?stored_fields=tags,counter
```

上述 get 操作的结果是：

``` json
{
   "_index": "twitter",
   "_type": "_doc",
   "_id": "1",
   "_version": 1,
   "found": true,
   "fields": {
      "tags": [
         "red"
      ]
   }
}
```

从文档本身获取的字段值始终作为数组返回。由于未存储 counter 字段，因此 get 请求在尝试获取 stored_field 时会忽略它。

也可以检索像 `_routing` 字段这样的元数据字段：

``` json
PUT twitter/_doc/2?routing=user1
{
    "counter" : 1,
    "tags" : ["white"]
}
```

```
GET twitter/_doc/2?routing=user1&stored_fields=tags,counter
```

上述 get 操作的结果是：

``` json
{
   "_index": "twitter",
   "_type": "_doc",
   "_id": "2",
   "_version": 1,
   "_routing": "user1",
   "found": true,
   "fields": {
      "tags": [
         "white"
      ]
   }
}
```

此外，只能通过 `stored_field` 选项返回叶子字段。因此无法返回对象字段，此类请求将失败。

## 直接获取 _source

使用 `/{index}/{type}/{id}/_source` 端点只获取文档的 `_source` 字段，而不包含任何其他内容。例如：

``` json
GET twitter/_doc/1/_source
```

您还可以使用相同的源过滤参数来控制将返回 `_source` 的哪些部分：

``` json
GET twitter/_doc/1/_source?_source_include=*.id&_source_exclude=entities
```

注意，_source 端点还有一个 HEAD 变体，可以有效地测试 document _source 的存在。如果在映射中禁用了现有文档，则该文档将没有 _source。

``` json
HEAD twitter/_doc/1/_source
```

## 路由

使用控制路由的能力进行索引时，为了获取文档，还应提供路由值。例如：

``` json
GET twitter/_doc/2?routing=user1
```

以上将获得 id 为 2 的推文，但将根据用户进行路由。注意，在没有正确路由的情况下发出 get，将导致无法获取文档。

## 首选项

控制哪些分片副本执行 get 请求的首选项。默认情况下，操作在分片复制副本之间随机化。

首选项可以设置为：

* _primary  
    该操作将仅在主分片上执行。
* _local  
    如果可能，操作将优先在本地分配的分片上执行。
* 自定义（string）值  
    自定义值将用于保证相同的分片将用于相同的自定义值。当在不同的刷新状态下击中不同的分片时，这可以帮助“跳跃值”。示例值可以是 Web 会话 ID 或用户名。

# Delete API

delete API 允许根据其 id 从特定索引中删除类型化的 JSON 文档。以下示例从名为 twitter 的索引中删除 JSON 文档，该索引名为 _doc，ID 为 1：

``` json
DELETE /twitter/_doc/1
```

上述删除操作的结果是：

``` json
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 2,
    "_primary_term": 1,
    "_seq_no": 5,
    "result": "deleted"
}
```

## 版本

索引的每个文档都是版本化的。删除文档时，version可以指定以确保我们尝试删除的相关文档实际上已被删除，并且在此期间没有更改。对文档执行的每个写入操作（包括删除）都会导致其版本递增。删除文档的版本号在删除后仍可短时间使用，以便控制并发操作。已删除文档的版本保持可用的时间长度由index.gc_deletes索引设置决定，默认为60秒。

## 路由

使用控制路由的能力进行索引时，为了删除文档，还应提供路由值。例如：

```
DELETE /twitter/_doc/1?routing=kimchy
```

以上将删除id为1的推文，但将根据用户进行路由。请注意，在没有正确路由的情况下发出删除将导致文档不被删除。

当 `_routing` 映射根据需要设置且未指定路由值时，delete api 将抛出 `RoutingMissingException` 并拒绝该请求。

## 超时

``` json
DELETE /twitter/_doc/1?timeout=5m
```

# Delete By Query API

最简单的 `_delete_by_query` 用法只会对匹配查询的每个文档执行删除操作。这是 API：

``` json
POST twitter/_delete_by_query
{
  "query": { 
    "match": {
      "message": "some message"
    }
  }
}
```

必须以与 Search API 相同的方式将查询作为值传递给查询键。您也可以使用与搜索 API 相同的 q 参数。

这将返回如下内容：

``` json
{
  "took" : 147,
  "timed_out": false,
  "deleted": 119,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1.0,
  "throttled_until_millis": 0,
  "total": 119,
  "failures" : [ ]
}
```

`_delete_by_query` 在启动时获取索引的快照，并使用内部版本控制删除它找到的内容。这意味着如果文档在拍摄快照的时间和处理删除请求之间发生更改，则会出现版本冲突。当版本匹配时，文档将被删除。

?> 由于内部版本控制不支持将值 0 作为有效版本号，因此无法使用 _delete_by_query 删除版本等于零的文档，并且将使请求失败。

在 `_delete_by_query` 执行期间，顺序执行多个搜索请求以查找要删除的所有匹配文档。每次找到一批文档时，都会执行相应的批量请求以删除所有这些文档。如果搜索或批量请求被拒绝， `_delete_by_query` 依赖于默认策略来重试被拒绝的请求（最多10次，指数退回）。达到最大重试次数限制会导致 `_delete_by_query` 中止，并且在响应失败时返回所有失败。已执行的删除仍然有效。换句话说，该过程不会回滚，只会中止。当第一个故障导致中止时，失败的批量请求返回的所有故障都将在 `failure` 元素中返回;因此，可能存在相当多的失败实体。

如果你想计算版本冲突而不是让它们中止，那么请在 url 中设置 `conflicts=proceed` 或在请求 body 中 设置 "conflicts": "proceed"。

回到 API 格式，这将删除 twitter 索引中的推文：

``` json
POST twitter/_doc/_delete_by_query?conflicts=proceed
{
  "query": {
    "match_all": {}
  }
}
```

也可以一次删除多个索引和多个类型的文档，就像搜索 API 一样：

``` json
POST twitter,blog/_docs,post/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}
```

如果提供路由，则路由将复制到滚动查询，将进程限制为与该路由值匹配的分片：

``` json
POST twitter/_delete_by_query?routing=1
{
  "query": {
    "range" : {
        "age" : {
           "gte" : 10
        }
    }
  }
}
```

默认情况下， `_delete_by_query` 使用 `1000` 的滚动批次。您可以使用 scroll_size URL 参数更改批量大小：

``` json
POST twitter/_delete_by_query?scroll_size=5000
{
  "query": {
    "term": {
      "user": "kimchy"
    }
  }
}
```

除了像pretty这样的标准参数之外，Delete By Query API还支持refresh，wait_for_completion，wait_for_active_shards，timeout和scroll。

发送 refresh 将刷新请求完成后按查询删除所涉及的所有分片。这与 Delete API 的 refresh 参数不同，后者只会刷新收到删除请求的分片。

## Response body

JSON响应如下所示：

``` json
{
  "took" : 147,
  "timed_out": false,
  "total": 119,
  "deleted": 119,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1.0,
  "throttled_until_millis": 0,
  "failures" : [ ]
}
```

* `took`: 整个操作从开始到结束的毫秒数。
* `timed_out`: 如果在查询执行删除期间执行的任何请求超时，则此标志设置为 true。
* `total`: 已成功处理的文档数。
* `deleted`: 已成功删除的文档数。
* `batches`: 通过查询删除拉回的滚动响应数。
* `version_conflicts`: 按查询删除的版本冲突数。
* `noops`: 对于按查询删除，此字段始终等于零。它只存在，以便通过查询删除，按查询更新和reindex API 返回具有相同结构的响应。
* `retries`: 通过查询删除尝试的重试次数。 bulk是重试的批量操作数，search是重试的搜索操作数。
* `throttled_millis`: 请求睡眠以符合 requests_per_second 的毫秒数。
* `requests_per_second`: 在通过查询删除期间有效执行的每秒请求数。
* `throttled_until_millis`: 在按查询响应删除时，此字段应始终等于零。它只在使用 Task API时有意义，它指示下一次（自纪元以来的毫秒），将再次执行受限制的请求以符合requests_per_second。
* `failures`: 如果在此过程中存在任何不可恢复的错误，则会出现故障数组。如果这是非空的，那么请求因为那些失败而中止。逐个查询是使用批处理实现的，任何故障都会导致整个进程中止，但当前批处理中的所有故障都会被收集到数组中。您可以使用conflict选项来防止reindex在版本冲突中中止。

# Update API

更新 API 允许基于提供的脚本更新文档。该操作从索引获取文档（与分片并置），运行脚本（使用可选的脚本语言和参数），并对结果进行索引（也允许删除或忽略操作）。它使用版本控制来确保在“get”和“reindex”期间没有发生更新。

请注意，此操作仍然意味着文档的完全重新索引，它只是删除了一些网络往返，并减少了get和索引之间版本冲突的可能性。需要启用 `_source` 字段才能使此功能正常工作。

例如，让我们索引一个简单的文档：

``` json
PUT test/_doc/1
{
    "counter" : 1,
    "tags" : ["red"]
}
```

## 脚本更新

现在，我们可以执行一个增加计数器的脚本：

``` json
POST test/_doc/1/_update
{
    "script" : {
        "source": "ctx._source.counter += params.count",
        "lang": "painless",
        "params" : {
            "count" : 4
        }
    }
}
```

我们可以在标签列表中添加一个标签（注意，如果标签存在，它仍会添加它，因为它是一个列表）：

``` json
POST test/_doc/1/_update
{
    "script" : {
        "source": "ctx._source.tags.add(params.tag)",
        "lang": "painless",
        "params" : {
            "tag" : "blue"
        }
    }
}
```

除了 `_source` 之外，还可以通过 ctx map 获得以下变量：_index，_type，_id，_version，_routing 和 _now（当前时间戳）。

我们还可以在文档中添加一个新字段：

``` json
POST test/_doc/1/_update
{
    "script" : "ctx._source.new_field = 'value_of_new_field'"
}
```

或者从文档中删除字段：

``` json
POST test/_doc/1/_update
{
    "script" : "ctx._source.remove('new_field')"
}
```

而且，我们甚至可以改变执行的操作。如果标签字段包含绿色，此示例将删除doc，否则它不执行任何操作（noop）：

``` json
POST test/_doc/1/_update
{
    "script" : {
        "source": "if (ctx._source.tags.contains(params.tag)) { ctx.op = 'delete' } else { ctx.op = 'none' }",
        "lang": "painless",
        "params" : {
            "tag" : "green"
        }
    }
}
```

## Updates with a partial document

更新 API 还支持传递部分文档，该部分文档将合并到现有文档中（简单的递归合并，对象的内部合并，替换核心“键/值”和数组）。要完全替换现有文档，应使用 index API。以下部分更新会向现有文档添加新字段：

``` json
POST test/_doc/1/_update
{
    "doc" : {
        "name" : "new_name"
    }
}
```

如果同时指定了 doc 和 script，则会忽略 doc。最好是将部分文档的字段对放在脚本本身中。

## Detecting noop updates

如果指定了 doc，则其值将与现有 _source 合并。默认情况下，不更改任何内容的更新会检测到它们没有更改任何内容并返回 “result”：“noop”，如下所示：

``` json
POST test/_doc/1/_update
{
    "doc" : {
        "name" : "new_name"
    }
}
```

如果在发送请求之前 name 是 new_name，则忽略整个更新请求。如果忽略请求，响应中的 result 元素将返回 noop。

``` json
{
   "_shards": {
        "total": 0,
        "successful": 0,
        "failed": 0
   },
   "_index": "test",
   "_type": "_doc",
   "_id": "1",
   "_version": 6,
   "result": "noop"
}
```

您可以通过设置 "detect_noop": false 来禁用此行为,如下所示：

``` json
POST test/_doc/1/_update
{
    "doc" : {
        "name" : "new_name"
    },
    "detect_noop": false
}
```

## Upserts

如果文档尚不存在，则 upsert 元素的内容将作为新文档插入。如果文档确实存在，那么将执行脚本：

``` json
POST test/_doc/1/_update
{
    "script" : {
        "source": "ctx._source.counter += params.count",
        "lang": "painless",
        "params" : {
            "count" : 4
        }
    },
    "upsert" : {
        "counter" : 1
    }
}
```

`scripted_upsert`

如果您希望脚本运行，无论文档是否存在 - 即脚本处理初始化文档而不是 upsert 元素 - 请将 scripted_upsert 设置为 true：

``` json
POST sessions/session/dh3sgudg8gsrgl/_update
{
    "scripted_upsert":true,
    "script" : {
        "id": "my_web_session_summariser",
        "params" : {
            "pageViewEvent" : {
                "url":"foo.com/bar",
                "response":404,
                "time":"2014-01-01 12:32"
            }
        }
    },
    "upsert" : {}
}
```

`doc_as_upsert`

将 doc_as_upsert 设置为 true 将使用 doc 的内容作为 upsert 值，而不是发送部分 doc 加上 upsert 文档：

``` json
POST test/_doc/1/_update
{
    "doc" : {
        "name" : "new_name"
    },
    "doc_as_upsert" : true
}
```

## 参数

更新操作支持以下查询字符串参数：

* `retry_on_conflict`: 在更新的 get 和 indexing 阶段之间，另一个进程可能已经更新了同一文档。默认情况下，更新将因版本冲突异常而失败。 `retry_on_conflict` 参数控制在最终抛出异常之前重试更新的次数。
* `routing`: 路由用于将更新请求路由到正确的分片，并在更新的文档不存在时为 upsert 请求设置路由。不能用于更新现有文档的路由。
* `timeout`: 超时等待分片变为可用。
* `wait_for_active_shards`: 在继续更新操作之前需要处于活动状态的分片副本数。
* `refresh`: 控制何时此请求所做的更改对搜索可见。
* `_source`: 允许控制是否以及如何在响应中返回更新的源。默认情况下，不会返回更新的源。请参阅源过滤了解详细信息
* `version`: 更新 API 在内部使用 Elasticsearch 的版本控制支持，以确保在更新期间文档不会更改。您可以使用 version 参数指定仅在文档版本与指定版本匹配时才更新文档。

# Update By Query API

_update_by_query 的最简单用法只是对索引中的每个文档执行更新而不更改源。这对于获取新属性或其他一些在线映射更改很有用。这是 API：

``` json
POST twitter/_update_by_query?conflicts=proceed
```

这将返回如下内容：

``` json
{
  "took" : 147,
  "timed_out": false,
  "updated": 120,
  "deleted": 0,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1.0,
  "throttled_until_millis": 0,
  "total": 120,
  "failures" : [ ]
}
```

_update_by_query 在索引启动时获取索引的快照，并使用内部版本控制索引它的内容。这意味着如果文档在拍摄快照的时间和处理索引请求之间发生更改，则会出现版本冲突。当版本匹配时，文档会更新，版本号会递增。

?> 由于内部版本控制不支持将值0作为有效版本号，因此无法使用_update_by_query更新版本等于零的文档，并且将使请求失败。

您还可以使用查询DSL限制 _update_by_query。这将更新用户 kimchy 的 twitter 索引中的所有文档：

``` json
POST twitter/_update_by_query?conflicts=proceed
{
  "query": { 
    "term": {
      "user": "kimchy"
    }
  }
}
```

必须以与 Search API 相同的方式将查询作为值传递给查询键。您也可以使用与搜索 API 相同的 q 参数。

到目前为止，我们只是在不更改文档来源的情况下更新文档。这对于拾取新属性等事情非常有用，但这只是其中一半的乐趣。_update_by_query 支持脚本来更新文档。这将增加所有kimchy的推文上的likes字段：

``` json
POST twitter/_update_by_query
{
  "script": {
    "source": "ctx._source.likes++",
    "lang": "painless"
  },
  "query": {
    "term": {
      "user": "kimchy"
    }
  }
}
```

就像在 Update API中一样，您可以设置 ctx.op 来更改执行的操作：

* `noop`: 如果您的脚本确定它不需要进行任何更改，请设置 ctx.op=“noop”。这将导致_update_by_query 从其更新中省略该文档。这种无操作将在响应 body 的 noop 计数器中报告。
* `delete`: 如果脚本决定必须删除文档，请设置 ctx.op=“delete”。删除将在响应正文中的已删除计数器中报告。

将 ctx.op 设置为其他任何内容都是错误的。在 ctx 中设置任何其他字段是错误的。

请注意，我们停止指定 conflict=proceed。在这种情况下，我们希望版本冲突中止该过程，以便我们可以处理失败。

此 API 不允许您移动它接触的文档，只需修改它们的源。这是故意的！我们没有规定将文档从原始位置删除。

也可以同时在多个索引和多个类型上完成这一切，就像搜索API一样：

```
POST twitter,blog/_doc,post/_update_by_query
```

如果提供路由，则路由将复制到滚动查询，将进程限制为与该路由值匹配的分片：

```
POST twitter/_update_by_query?routing=1
```

默认情况下 _update_by_query 使用 1000 的滚动批次。您可以使用 scroll_size URL 参数更改批量大小：

```
POST twitter/_update_by_query?scroll_size=100
```

_update_by_query也可以通过指定这样的管道来使用“Ingest Node”功能：

``` json
PUT _ingest/pipeline/set-foo
{
  "description" : "sets foo",
  "processors" : [ {
      "set" : {
        "field": "foo",
        "value": "bar"
      }
  } ]
}
POST twitter/_update_by_query?pipeline=set-foo
```

# Multi Get API

Multi GET API 允许基于索引，类型（可选）和 id（以及可能的路由）获取多个文档。响应包括 docs 数组，其中所有获取的文档按顺序对应于原始的 multi-get 请求（如果特定 get 的失败，则包含此错误的对象将包含在响应中）。成功获取的结构在结构上类似于 get API 提供的文档。

``` json
GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "1"
        },
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "2"
        }
    ]
}
```

mget 端点也可以用于索引（在这种情况下，它在主体中不是必需的）：

``` json
GET /test/_mget
{
    "docs" : [
        {
            "_type" : "_doc",
            "_id" : "1"
        },
        {
            "_type" : "_doc",
            "_id" : "2"
        }
    ]
}
```

或者

``` json
GET /test/type/_mget
{
    "docs" : [
        {
            "_id" : "1"
        },
        {
            "_id" : "2"
        }
    ]
}
```

在这种情况下，可以直接使用 ids 元素来简化请求：

``` json
GET /test/type/_mget
{
    "ids" : ["1", "2"]
}
```

## Source filtering

默认情况下，将为每个文档（如果存储）返回 _source 字段。与 get API 类似，您只能使用 _source 参数检索 _source 的一部分（或根本不检索）。您还可以使用 url 参数 _source，_source_include＆_source_exclude 来指定默认值，这些默认值将在没有按文档说明时使用。

例如：

``` json
GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "1",
            "_source" : false
        },
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "2",
            "_source" : ["field3", "field4"]
        },
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "3",
            "_source" : {
                "include": ["user"],
                "exclude": ["user.location"]
            }
        }
    ]
}
```

## Fields

可以指定要为每个文档检索特定存储的字段以获取，类似于 Get API 的 stored_fields 参数。例如：

``` json
GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "1",
            "stored_fields" : ["field1", "field2"]
        },
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "2",
            "stored_fields" : ["field3", "field4"]
        }
    ]
}
```

或者，您可以将查询字符串中的 stored_fields 参数指定为默认值，以应用于所有文档。

``` json
GET /test/type/_mget?stored_fields=field1,field2
{
    "docs" : [
        {
            "_id" : "1" 
        },
        {
            "_id" : "2",
            "stored_fields" : ["field3", "field4"] 
        }
    ]
}
```

## Routing

您还可以将路由值指定为参数：

``` json
GET /_mget?routing=key1
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "1",
            "routing" : "key2"
        },
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "2"
        }
    ]
}
```

在这个例子中，文档 `test/type/2` 将从对应于路由密钥 key1 的分片中获取，但是文档 `test/type/1` 将从对应于路由密钥 key2 的分片中获取。

# Bulk API

批量 API 使得在单个 API 调用中执行许多索引/删除操作成为可能。这可以大大提高索引速度。

REST API 端点是 `/_bulk` ，它期望以下换行符分隔 JSON（NDJSON）结构：

```
action_and_meta_data\n
optional_source\n
action_and_meta_data\n
optional_source\n
....
action_and_meta_data\n
optional_source\n
```

!> 注意：最后一行数据必须以换行符 `\n` 结尾。每个换行符前面都可以有回车符 `\r\n`。向此端点发送请求时，Content-Type 标头应设置为 `application/x-ndjson` 。

可能的操作是索引，创建，删除和更新。 index和create期望下一行的源，并且具有与标准索引API的op_type参数相同的语义（即如果已存在具有相同索引和类型的文档，则create将失败，而index将根据需要添加或替换文档）。delete 不会指望以下行中的源，并且具有与标准删除 API 相同的语义。update 期望在下一行指定部分 doc，upsert 和 script 及其选项。

如果要为 curl 提供文本文件输入，则必须使用 --data-binary 标志而不是 plain -d。后者不保留换行符。例：

``` json
$ cat requests
{ "index" : { "_index" : "test", "_type" : "_doc", "_id" : "1" } }
{ "field1" : "value1" }
$ curl -s -H "Content-Type: application/x-ndjson" -XPOST localhost:9200/_bulk --data-binary "@requests"; echo
{"took":7, "errors": false, "items":[{"index":{"_index":"test","_type":"_doc","_id":"1","_version":1,"result":"created","forced_refresh":false}}]}
```

因为这种格式使用文字 `\n` 作为分隔符，所以请确保 JSON 操作和源不是很漂亮。以下是批量命令的正确序列示例：

``` json
POST _bulk
{ "index" : { "_index" : "test", "_type" : "_doc", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_type" : "_doc", "_id" : "2" } }
{ "create" : { "_index" : "test", "_type" : "_doc", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_type" : "_doc", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }
```

批量操作的结果是：

``` json
{
   "took": 30,
   "errors": false,
   "items": [
      {
         "index": {
            "_index": "test",
            "_type": "_doc",
            "_id": "1",
            "_version": 1,
            "result": "created",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "status": 201,
            "_seq_no" : 0,
            "_primary_term": 1
         }
      },
      {
         "delete": {
            "_index": "test",
            "_type": "_doc",
            "_id": "2",
            "_version": 1,
            "result": "not_found",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "status": 404,
            "_seq_no" : 1,
            "_primary_term" : 2
         }
      },
      {
         "create": {
            "_index": "test",
            "_type": "_doc",
            "_id": "3",
            "_version": 1,
            "result": "created",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "status": 201,
            "_seq_no" : 2,
            "_primary_term" : 3
         }
      },
      {
         "update": {
            "_index": "test",
            "_type": "_doc",
            "_id": "1",
            "_version": 2,
            "result": "updated",
            "_shards": {
                "total": 2,
                "successful": 1,
                "failed": 0
            },
            "status": 200,
            "_seq_no" : 3,
            "_primary_term" : 4
         }
      }
   ]
}
```

## Update

使用更新操作时，retry_on_conflict可用作操作本身的字段（不在额外的有效负载行中），以指定在版本冲突的情况下应重试更新的次数。

更新操作有效内容支持以下选项：doc（部分文档），upsert，doc_as_upsert，script，params（用于脚本），lang（用于脚本）和_source。

``` json
POST _bulk
{ "update" : {"_id" : "1", "_type" : "_doc", "_index" : "index1", "retry_on_conflict" : 3} }
{ "doc" : {"field" : "value"} }
{ "update" : { "_id" : "0", "_type" : "_doc", "_index" : "index1", "retry_on_conflict" : 3} }
{ "script" : { "source": "ctx._source.counter += params.param1", "lang" : "painless", "params" : {"param1" : 1}}, "upsert" : {"counter" : 1}}
{ "update" : {"_id" : "2", "_type" : "_doc", "_index" : "index1", "retry_on_conflict" : 3} }
{ "doc" : {"field" : "value"}, "doc_as_upsert" : true }
{ "update" : {"_id" : "3", "_type" : "_doc", "_index" : "index1", "_source" : true} }
{ "doc" : {"field" : "value"} }
{ "update" : {"_id" : "4", "_type" : "_doc", "_index" : "index1"} }
{ "doc" : {"field" : "value"}, "_source": true}
```

# Reindex API

https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html