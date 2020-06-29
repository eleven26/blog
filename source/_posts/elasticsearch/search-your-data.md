---
title: Elasticsearch - 搜索你的数据
date: 2020-06-29 15:39:04
tags: [ElasticSearch]
---

## 运行搜索

你可以使用[搜索API](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html)搜索 Elasticsearch 数据流或索引中存储的数据。

该 API 可以运行两种类型的搜索，具体取决于你如何提供[查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-your-data.html#search-query)：

* [URI 搜索](https://www.elastic.co/guide/en/elasticsearch/reference/master/run-a-search.html#run-uri-search)
    * 通过查询参数提供查询。URI 搜索往往更简单，最适合测试。
* [请求体搜索](https://www.elastic.co/guide/en/elasticsearch/reference/master/run-a-search.html#run-request-body-search)
    * 通过 API 请求的 JSON body 提供查询。这些查询是用 [查询DSL](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html) 编写的。我们建议在大多数生产用例中使用请求体搜索。

> 如果同时在 URI 和请求正文中指定查询，则搜索 API 请求仅运行 URI 查询。


### 运行 URI 搜索

你可以使用搜索 API 的 [q查询字符串参数](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#search-api-query-params-q) 在请求的 URI 中运行搜索。该 q 参数仅接受以 Lucene 的[查询字符串语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html#query-string-syntax)编写的查询。

例子:

首先，向 Elasticsearch 索引添加一些数据。

以下 [bulk API](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-bulk.html) 将一些示例用户日志数据添加到 `user_logs_00001` 索引。

<details>
    <summary>示例</summary>
    
    $response = $client->bulk([
        'body' => [
            [
                'index' => [
                    '_index' => 'user_logs_00001',
                    '_id' => 1,
                ],
            ],
            [
                '@timestamp' => '2020-12-06T11:04:05.000Z',
                'user' => [
                    'id' => 'vlb44hny',
                ],
                'message' => 'Login attempt failed',
            ],
    
            [
                'index' => [
                    '_index' => 'user_logs_00001',
                    '_id' => 2,
                ],
            ],
            [
                '@timestamp' => '2020-12-07T11:06:07.000Z',
                'user' => [
                    'id' => '8a4f500d',
                ],
                'message' => 'Login successful',
            ],
    
            [
                'index' => [
                    '_index' => 'user_logs_00001',
                    '_id' => 3,
                ],
            ],
            [
                '@timestamp' => '2020-12-07T11:07:08.000Z',
                'user' => [
                    'id' => 'l7gk7f82',
                ],
                'message' => 'Logout successful',
            ],
        ],
    ]);
    
    echo json_encode($response, JSON_PRETTY_PRINT);
</details>

现在你可以使用 URI 搜索来匹配一个 `user.id` 为 `l7gk7f82` 的文档，请注意查询通过 `q` 查询参数来指定。

```
$response = $client->search([
    'index' => 'user_logs_00001',
    'q' => 'user.id:8a4f500d',
]);

echo json_encode($response, JSON_PRETTY_PRINT);
```

响应结果的 `hits.hits` 属性包含了匹配到的文档。


### 运行一个请求体查询

你也可以通过 [查询DSL](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html) 语法来传递一个 [query](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#request-body-search-query) 请求体参数进行查询。


<details>
    <summary>示例</summary>
    
    $response = $client->search([
        'index' => 'user_logs_00001',
        'body' => [
            'query' => [
                'match' => [
                    'message' => '登录成功',
                ],
            ],
        ]
    ]);
    
    echo json_encode($response, JSON_PRETTY_PRINT);
</details>

API 返回以下响应：

该 `hits.hits` 属性包含匹配的文档。默认情况下，响应按这些匹配的文档 `_score` 的相关性得分排序，该得分衡量每个文档与查询的匹配程度。

<details>
    <summary>响应</summary>

    {
        "took": 1,
        "timed_out": false,
        "_shards": {
            "total": 1,
            "successful": 1,
            "skipped": 0,
            "failed": 0
        },
        "hits": {
            "total": {
                "value": 3,
                "relation": "eq"
            },
            "max_score": 0.9983525,
            "hits": [
                {
                    "_index": "user_logs_00001",
                    "_type": "_doc",
                    "_id": "2",
                    "_score": 0.9983525,
                    "_source": {
                        "@timestamp": "2020-12-07T11:06:07.000Z",
                        "user": {
                            "id": "8a4f500d"
                        },
                        "message": "Login successful"
                    }
                },
                {
                    "_index": "user_logs_00001",
                    "_type": "_doc",
                    "_id": "3",
                    "_score": 0.49917626,
                    "_source": {
                        "@timestamp": "2020-12-07T11:07:08.000Z",
                        "user": {
                            "id": "l7gk7f82"
                        },
                        "message": "Logout successful"
                    }
                },
                {
                    "_index": "user_logs_00001",
                    "_type": "_doc",
                    "_id": "1",
                    "_score": 0.42081726,
                    "_source": {
                        "@timestamp": "2020-12-06T11:04:05.000Z",
                        "user": {
                            "id": "vlb44hny"
                        },
                        "message": "Login attempt failed"
                    }
                }
            ]
        }
    }
</details>


### 搜索多个数据流和索引

要搜索多个数据流和索引，请将它们作为逗号分隔的值添加到搜索 API 请求路径中。

<details>
    <summary>示例</summary>
    以下请求搜索 `user_logs_00001` 和 `user_logs_00002` 索引。
    
    $response = $client->search([
        'index' => 'user_logs_00001,user_logs_00002',
        'body' => [
            'query' => [
                'match' => [
                    'message' => 'Login Successful',
                ],
            ],
        ]
    ]);
    
    echo json_encode($response, JSON_PRETTY_PRINT);
</details>

你也可以使用通配符 `*` 模式搜索多个数据流和索引。

<details>
    <summary>示例</summary>
    
    $response = $client->search([
        'index' => 'user_logs*',
        'body' => [
            'query' => [
                'match' => [
                    'message' => 'Login Successful',
                ],
            ],
        ]
    ]);
    
    echo json_encode($response, JSON_PRETTY_PRINT);
</details>

要搜索集群中的所有数据流和索引，请从请求路径中省略目标。或者，你可以使用 `_all` 或 `*`。

<details>
    <summary>示例</summary>
    
    $response = $client->search([
        'index' => '',         // 搜索全部索引
    //    'index' => '_all',   // 搜索全部索引
    //    'index' => '*',      // 搜索全部索引
        'body' => [
            'query' => [
                'match' => [
                    'message' => 'Login Successful',
                ],
            ],
        ]
    ]);
    
    echo json_encode($response, JSON_PRETTY_PRINT);
</details>


### 分页搜索结果

默认情况下，搜索 API 返回前 10 个匹配的文档。

要分页显示更多结果，可以使用搜索 API `size` 和 `from` 参数。该 `size` 参数是要返回匹配文档的数量。该 `from` 是从完整结果集开始的零索引偏移量，该偏移量指示要开始使用的文档。

<details>
    <summary>示例</summary>
    以下搜索 API 请求将 `from` 偏移量设置为 5，表示请求偏移量或跳过前五个匹配文档。
    该 `size` 参数是 20，这意味着该请求最多可返回 20 个文档，开始偏移。
    
    $response = $client->search([
        'index' => '*',
        'body' => [
            'query' => [
                'term' => [
                    'user.id' => '8a4f500d',
                ],
            ],
        ],
        'from' => 5,
        'size' => 20,
    ]);
    
    echo json_encode($response, JSON_PRETTY_PRINT);
</details>

默认情况下，你不能使用 `from` 和 `size` 参数分页浏览超过 10000 个文档。使用 `index.max_result_window` 索引设置来设置此限制。

深度分页或一次请求许多结果可能会导致搜索缓慢。结果在返回之前先进行排序。由于搜索请求通常跨越多个分片，因此每个分片必须生成自己的排序结果。然后必须对这些单独的结果进行合并和排序，以确保总体排序顺序正确。

作为深度分页的替代方法，我们建议使用 `scroll` 或 `search_after` 参数。

> Elasticsearch 使用 Lucene 的内部文档 ID 作为平局。这些内部文档 ID 在相同数据的副本之间可能完全不同。在进行分页时，您可能偶尔会看到排序顺序相同的文档的顺序不一致。


### 检索选定的字段

默认情况下，搜索响应中的每个 `_source` 匹配都包括 document，这是对文档建立索引时提供的整个 JSON 对象。如果在搜索响应中仅需要某些源字段，则可以使用 `source-filtering` 来限制返回源的哪些部分。

仅使用文档源返回字段有一些限制：

* 该 `_source` 不包含多字段或字段别名。同样，源中的字段也不包含使用 `copy_to` 映射参数复制的值。
* 由于 `_source` 在 Lucene 中存储为单个字段，因此即使只需要少量字段，也必须加载和解析整个源对象。

为了避免这些限制，你可以：

* 使用 `docvalue_fields` 参数获取选定字段的值。当返回相当少量的支持 doc 值的字段（例如关键字和日期）时，这是一个不错的选择。
* 使用 `sorted_fields` 参数获取特定存储字段的值。（使用 `store` 映射选项的字段。）

你可以在以下各节中找到有关这些方法的更多详细信息：

* 源过滤
* 文件值栏位
* 储存栏位


#### 源过滤

你可以使用该 `_source` 参数选择返回源的哪些字段。这称为源过滤。

如下的搜索请求设置 `_source` 请求体参数为 `false`，这样请求结果里就不会包含 `_source` 字段。

<details>
    <summary>示例</summary>
   
    $response = $client->search([
        'index' => '*',
        'body' => [
            'query' => [
                'term' => [
                    'user.id' => '8a4f500d',
                ],
            ],
        ],
        '_source' => false,
    ]);
    
    echo json_encode($response, JSON_PRETTY_PRINT);
</details>

也可以通过 `*` 通配符来让搜索 API 返回对应的字段，下面的请求返回的响应结果中的 `_source` 字段只会包含 `obj` 字段以及它的属性：

<details>
    <summary>示例</summary>
   
    $response = $client->search([
        'index' => '*',
        'body' => [
            'query' => [
                'term' => [
                    'user.id' => '8a4f500d',
                ],
            ],
        ],
        '_source' => 'obj.*',
    ]);
    
    echo json_encode($response, JSON_PRETTY_PRINT);
</details>

你也可以通过数组指定多个字段名匹配的模式，如下的请求会返回 `obj1` 和 `obj2` 字段和它们的属性：

<details>
    <summary>示例</summary>
   
    $response = $client->search([
        'index' => '*',
        'body' => [
            'query' => [
                'term' => [
                    'user.id' => '8a4f500d',
                ],
            ],
        ],
        '_source' => ['obj1.*', 'obj2.*'],
    ]);
    
    echo json_encode($response, JSON_PRETTY_PRINT);
</details>

为了更好地控制，你可以指定一个对象，该对象在参数中包含 `includes` 和 `excludes` 模式的数组 `_source`。

如果 `includes` 指定了属性，则仅返回与其模式之一匹配的源字段。你可以使用 `excludes` 属性从此子集中排除字段。

如果 `includes` 未指定该属性，则返回整个文档源，不包括与该 `excludes` 属性中的模式匹配的任何字段。

以下搜索 API 请求仅返回 `obj1` 和 `obj2` 字段及属性的源，不包括任何子 `description` 字段。

<details>
    <summary>示例</summary>
    
    $response = $client->search([
        'index' => '*',
        'body' => [
            'query' => [
                'term' => [
                    'user.id' => '8a4f500d',
                ],
            ],
        ],
        '_source' => [
            'includes' => [
                'obj1.*',
                'obj2.*',
            ],
            'excludes' => [
                '*.description',
            ],
        ],
    ]);
    
    echo json_encode($response, JSON_PRETTY_PRINT);
</details>


#### `docvalue_fields`

你可以使用 `docvalue_fields` 参数返回搜索响应中一个或多个字段的 `doc-values`。

Doc 值存储与 `_source` 相同的值，但是在磁盘上基于列的结构中进行了优化，该结构针对排序和聚合进行了优化。由于每个字段都是单独存储的，因此 Elasticsearch 仅读取所请求的字段值，并且可以避免加载整个文档。

默认情况下，将为支持的字段存储文档值。但是，`text` 和 `text_annotated` 字段不支持 doc 值。

以下搜索请求使用该 `docvalue_fields` 参数来检索以下字段的 doc 值：
* 名称以 `my_ip` 开头的字段
* `my_keyword_field`
* 名称以 `_date_field` 结尾的字段
    
<details>
    <summary>例</summary>

    $response = $client->search([
        'index' => '*',
        'body' => [
            'query' => [
                'match_all' => (object)[
                ],
            ],
            'docvalue_fields' => [
                'my_ip*', 
                [
                    'field' => 'my_keyword_field',
                ],
                [
                    'field' => '*_date_field',
                    'format' => 'epoch_millis',
                ],
            ],
        ],
    ]);
    
    echo json_encode($response, JSON_PRETTY_PRINT);
</details>

- 通配符 patten，用于匹配以字符串形式指定的字段名称
- 通配符 patten，用于匹配指定为对象的字段名称
- 使用对象符号，可以使用 `format` 参数指定字段的返回 doc 值格式。日期字段支持日期格式，数值字段支持 DecimalFormat 模式。其他字段数据类型不支持该 format 参数。


#### 储存栏位

也可以使用 `store` 映射选项存储单个字段的值。你可以使用 `stored_fields` 参数将这些存储的值包括在搜索响应中。

