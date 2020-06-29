---
title: Elasticsearch 入门（二）- 开始搜索
date: 2020-06-29 11:47:01
tags: [ElasticSearch]
---

#### 不做任何过滤的搜索

一旦将一些数据摄取到 Elasticsearch 索引之后，你就可以通过 `/{index}/_search` 来发起搜索请求。要访问全套搜索功能，请使用 Elasticsearch Query DSL 在请求正文中指定搜索条件。您可以在请求 URI 中指定搜索的索引的名称。

例如，以下请求检索 `bank` 按账号排序的索引中的所有文档：

```
GET /bank/_search
{
    "query": { "match_all": {} },
    "sort": {
        { "account_number": "asc" }
    }
}
```

PHP 代码：

```PHP
$params = [
    'index' => 'bank',
    'body' => [
        'query' => [
            'match_all' => (object)[
            ],
        ],
        'sort' => [
            [
                'account_number' => 'asc',
            ],
        ],
    ],
];
$response = $client->search($params);
```

> `'match_all'` 后面如果是空数组，需要加上 `(object)` 转换成空对象。

默认情况下，返回值的 `hits` 字段包含了符合搜索条件的前十条文档：

```
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
    "total" : {
        "value": 1000,
        "relation": "eq"
    },
    "max_score" : null,
    "hits" : [ {
      "_index" : "bank",
      "_id" : "0",
      "sort": [0],
      "_score" : null,
      "_source" : {"account_number":0,"balance":16623,"firstname":"Bradshaw","lastname":"Mckenzie","age":29,"gender":"F","address":"244 Columbus Place","employer":"Euron","email":"bradshawmckenzie@euron.com","city":"Hobucken","state":"CO"}
    }, {
      "_index" : "bank",
      "_id" : "1",
      "sort": [1],
      "_score" : null,
      "_source" : {"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
    }, ...
    ]
  }
}
```

响应结果也提供了关于搜索请求的一些其他信息：

* `took`: 单位为 `ms`，Elasticsearch 处理这个请求用了多久
* `timed_out`: 搜索请求是否超时
* `_shards`: 搜索了多少个分片，以及成功、失败或跳过了多少个分片
* `max_score`: 找到的最相关文件的分数
* `hits.total.value`: 找到了多少个匹配的文档
* `hits.sort`: 文档的排序（不按相关性得分排序时）
* `hits._score`: 文档的相关性得分（使用时不适用 `match_all`）

每个请求是独立的：Elasticsearch 在请求中不维护任何状态信息。要翻阅搜索结果，请在您的请求中指定 `from` 和 `size` 参数。

例如，以下请求获取了第 10 到 19 条文档：

```PHP
$params = [
    'index' => 'bank',
    'body' => [
        'query' => [
            'match_all' => (object)[
            ],
        ],
        'sort' => [
            [
                'account_number' => 'asc',
            ],
        ],
        'from' => 10,
        'size' => 10,
    ],
];

$response = $client->search($params);
```

### 根据字段匹配搜索

#### 不精确匹配

既然已经了解了如何提交基本的搜索请求，则可以开始构建更有趣的 `match_all` 查询。

要在字段中搜索特定术语，可以使用 `match` 查询。例如，以下请求搜索该 `address` 字段以查找地址包含 `mill` 或 `lane` 的客户：

```PHP
$params = [
    'index' => 'bank',
    'body' => [
        'query' => [
            'match' => [
                'address' => 'mill lane',
            ],
        ],
    ],
];

$response = $client->search($params);
```


#### 精确匹配

如果要执行词组搜索而不是匹配单个词，请使用 `match_phrase` 代替 `match`。例如，以下请求仅匹配包含短语 `mill lane` 的地址：

```PHP
$params = [
    'index' => 'bank',
    'body' => [
        'query' => [
            'match_phrase' => [
                'address' => 'mill lane',
            ],
        ],
    ],
];

$response = $client->search($params);
```


要想构造更复杂的查询，可以使用 `bool` 查询来组合多个查询条件。你可以根据需要（必须匹配），期望（应该匹配）或不期望（必须不匹配）指定条件。

例如，以下请求在 `bank` 索引中搜索属于 40 岁客户的账户，但不包括住在爱达荷州（ID）的任何人：

```PHP
$params = [
    'index' => 'bank',
    'body' => [
        'query' => [
            'bool' => [
                'must' => [
                    'match' => [
                        'age' => '40',
                    ],
                ],
                'must_not' => [
                    [
                        'match' => [
                            'state' => 'ID',
                        ],
                    ],
                ],
            ],
        ],
    ],
];
$response = $client->search($params);
```

`bool` 查询中的每个 `must`，`should` 和 `must_not` 元素成为查询子句。文档满足每个 `must` 或 `should` 的标准的程度有助于文档的相关性得分。分数越高，文档就越符合你的搜索条件。默认情况下，Elasticsearch 返回按这些相关性分数排名的文档。

`must_not` 子句中的条件被视为过滤器。它影响文件是否包含在结果中，但不会影响文件的评分方式。您还可以显式指定任意过滤器，以基于结构化数据包括或排除文档。

例如，以下请求使用范围过滤器将结果限制为余额在 2000 美元到 30000 美元（含）之间的账户。

```PHP
$params = [
    'index' => 'bank',
    'body' => [
        'query' => [
            'bool' => [
                'must' => [
                    'match_all' => [
                    ],
                ],
                'filter' => [
                    'range' => [
                        'balance' => [
                            'gte' => 20000,
                            'lte' => 30000,
                        ],
                    ],
                ],
            ],
        ],
    ],
];

$response = $client->search($params);
```
