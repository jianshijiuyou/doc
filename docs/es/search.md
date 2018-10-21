# Query

搜索请求主体中的查询元素允许使用查询 DSL 定义查询。

``` json
GET /_search
{
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

# From / Size

可以使用 from 和 size 参数完成结果的分页。from 参数定义要获取的第一个结果的偏移量。size 参数允许您配置要返回的最大命中数。

虽然 from 和 size 可以设置为请求参数，但也可以在搜索主体中设置它们。from 默认值为 0，size 默认为 10。

``` json
GET /_search
{
    "from" : 0, "size" : 10,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

请注意，from + size 不能超过 index.max_result_window 索引设置，默认为 10000。有关更有效的深度滚动方法，请参阅 Scroll 或 Search After API。

# Sort

允许您在特定字段上添加一个或多个排序。每种类型也可以反转。排序是在每个字段级别定义的，_score 的特殊字段名称按分数排序，_doc 按索引顺序排序。

假设以下索引映射：

``` json
PUT /my_index
{
    "mappings": {
        "_doc": {
            "properties": {
                "post_date": { "type": "date" },
                "user": {
                    "type": "keyword"
                },
                "name": {
                    "type": "keyword"
                },
                "age": { "type": "integer" }
            }
        }
    }
}
```

``` json
GET /my_index/_search
{
    "sort" : [
        { "post_date" : {"order" : "asc"}},
        "user",
        { "name" : "desc" },
        { "age" : "desc" },
        "_score"
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

?> _doc除了是最有效的排序顺序之外没有真正的用例。因此，如果您不关心文档的返回顺序，那么您应该按_doc排序。滚动时尤其有用。

## Sort Values

返回的每个文档的排序值也作为响应的一部分返回。

## Sort Order

order 选项可以具有以下值：

* `asc`: 按升序排序
* `desc`: 按降序排序

在 _score 上排序时，顺序默认为 desc，在排序其他任何内容时默认为 asc。

## Sort mode option

Elasticsearch 支持按数组或多值字段进行排序。mode 选项控制选择哪个数组值以对其所属的文档进行排序。mode 选项可以具有以下值：

* `min`: 选择最低值。
* `max`: 选择最高值。
* `sum`: 使用所有值的总和作为排序值。仅适用于基于数字的数组字段。
* `avg`: 使用所有值的平均值作为排序值。仅适用于基于数字的数组字段。
* `median`: 使用所有值的中位数作为排序值。仅适用于基于数字的数组字段。

## Sort mode example usage

在下面的示例中，字段价格每个文档有多个价格。在这种情况下，结果点击将根据每个文档的平均价格按价格上升进行排序。

``` json
PUT /my_index/_doc/1?refresh
{
   "product": "chocolate",
   "price": [20, 4]
}

POST /_search
{
   "query" : {
      "term" : { "product" : "chocolate" }
   },
   "sort" : [
      {"price" : {"order" : "asc", "mode" : "avg"}}
   ]
}
```

# Source filtering

允许控制每次点击返回 _source 字段的方式。

默认情况下，操作将返回 _source 字段的内容，除非您已使用 stored_fields 参数或已禁用 _source 字段。

您可以使用 _source 参数关闭 _source 检索：

要禁用 _source 检索设置为 false：

``` json
GET /_search
{
    "_source": false,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

_source 还接受一个或多个通配符模式来控制应该返回 _source 的哪些部分：

例如：

``` json
GET /_search
{
    "_source": "obj.*",
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

或

``` json
GET /_search
{
    "_source": [ "obj1.*", "obj2.*" ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

最后，为了完全控制，您可以指定包含和排除模式：

``` json
GET /_search
{
    "_source": {
        "includes": [ "obj1.*", "obj2.*" ],
        "excludes": [ "*.description" ]
    },
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

# Fields

允许有选择地为搜索匹配所代表的每个文档加载特定的存储字段。

``` json
GET /_search
{
    "stored_fields" : ["user", "postDate"],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

`*` 可用于从文档加载所有存储的字段。

空数组将仅返回每个命中的 _id 和 _type ，例如：

``` json
GET /_search
{
    "stored_fields" : [],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

如果未存储请求的字段（ store 映射设置为 false），则将忽略它们。

从文档本身获取的存储字段值始终作为数组返回。相反，像 _routing 这样的元数据字段永远不会作为数组返回。

此外，只能通过字段选项返回叶字段。因此无法返回对象字段，此类请求将失败。

脚本字段也可以自动检测并用作字段，因此可以使用_source.obj1.field1之类的东西，但不推荐使用，因为obj1.field1也可以使用。

## 完全禁用存储的字段

要完全禁用存储的字段（和元数据字段），请使用：`_none_`：

``` json
GET /_search
{
    "stored_fields": "_none_",
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

# Script Fields

允许为每个匹配返回脚本评估（基于不同的字段），例如：

``` json
GET /_search
{
    "query" : {
        "match_all": {}
    },
    "script_fields" : {
        "test1" : {
            "script" : {
                "lang": "painless",
                "source": "doc['price'].value * 2"
            }
        },
        "test2" : {
            "script" : {
                "lang": "painless",
                "source": "doc['price'].value * params.factor",
                "params" : {
                    "factor"  : 2.0
                }
            }
        }
    }
}
```

脚本字段可以处理未存储的字段（在上述情况下为 my_field_name ），并允许返回要返回的自定义值（脚本的评估值）。


脚本字段还可以访问实际的 `_source` 文档，并使用 `params ['_source']` 提取要从中返回的特定元素。这是一个例子：

``` json
GET /_search
    {
        "query" : {
            "match_all": {}
        },
        "script_fields" : {
            "test1" : {
                "script" : "params['_source']['message']"
            }
        }
    }
```

注意这里的 _source 关键字来导航类似 json 的模型。

