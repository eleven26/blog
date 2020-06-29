---
title: Elasticsearch 入门（三）- 使用聚合分析结果
date: 2020-06-29 11:47:02
tags: [ElasticSearch]
---

Elasticsearch 聚合使你能够获取有关搜索结果的元信息，并回答诸如 “德克萨斯州有多少账户持有人？” 之类的问题。或 “田纳西州的平均账户余额是多少？” 你可以在一个请求中搜索文档，过滤匹配，并使用汇总分析结果。

例如，以下请求使用 `terms` 汇总将 `bank` 索引中的所有账户按状态分组，并按降序返回账户数量最多的十个州：

```PHP
$params = [
    'index' => 'bank',
    'body' => [
        'size' => 0,
        'aggs' => [
            'group_by_state' => [
                'terms' => [
                    'field' => 'state.keyword',
                ],
            ],
        ],
    ],
];
$response = $client->search($params);
```

响应结果中的 `buckets` 是 `state` 字段的值，`doc_count` 是每一个州的账户数量。例如，你可以看到有 27 个账户在 `ID`(Idaho)。因为请求设置了 `size=0`，响应仅仅包含了聚合结果。

```
{
    "took": 2,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1000,
            "relation": "eq"
        },
        "max_score": null,
        "hits": []
    },
    "aggregations": {
        "group_by_state": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 743,
            "buckets": [
                {
                    "key": "TX",
                    "doc_count": 30
                },
                {
                    "key": "MD",
                    "doc_count": 28
                },
                {
                    "key": "ID",
                    "doc_count": 27
                },
                {
                    "key": "AL",
                    "doc_count": 25
                },
                {
                    "key": "ME",
                    "doc_count": 25
                },
                {
                    "key": "TN",
                    "doc_count": 25
                },
                {
                    "key": "WY",
                    "doc_count": 25
                },
                {
                    "key": "DC",
                    "doc_count": 24
                },
                {
                    "key": "MA",
                    "doc_count": 24
                },
                {
                    "key": "ND",
                    "doc_count": 24
                }
            ]
        }
    }
}
```

你可以组合聚合以构建更复杂的数据汇总。例如，以下请求将一个 `avg` 聚合嵌套在先前的 `group_by_state` 聚合中，以计算每个状态的平均账户余额。

```PHP
$params = [
    'index' => 'bank',
    'body' => [
        'size' => 0,
        'aggs' => [
            'group_by_state' => [
                'terms' => [
                    'field' => 'state.keyword',
                ],
                'aggs' => [
                    'average_balance' => [
                        'avg' => [
                            'field' => 'balance',
                        ],
                    ],
                ],
            ],
        ],
    ],
];

$response = $client->search($params);
```

结果：

```
{
    "took": 19,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1000,
            "relation": "eq"
        },
        "max_score": null,
        "hits": []
    },
    "aggregations": {
        "group_by_state": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 743,
            "buckets": [
                {
                    "key": "TX",
                    "doc_count": 30,
                    "average_balance": {
                        "value": 26073.3
                    }
                },
                {
                    "key": "MD",
                    "doc_count": 28,
                    "average_balance": {
                        "value": 26161.535714285714
                    }
                },
                {
                    "key": "ID",
                    "doc_count": 27,
                    "average_balance": {
                        "value": 24368.777777777777
                    }
                },
                {
                    "key": "AL",
                    "doc_count": 25,
                    "average_balance": {
                        "value": 25739.56
                    }
                },
                {
                    "key": "ME",
                    "doc_count": 25,
                    "average_balance": {
                        "value": 21663
                    }
                },
                {
                    "key": "TN",
                    "doc_count": 25,
                    "average_balance": {
                        "value": 28365.4
                    }
                },
                {
                    "key": "WY",
                    "doc_count": 25,
                    "average_balance": {
                        "value": 21731.52
                    }
                },
                {
                    "key": "DC",
                    "doc_count": 24,
                    "average_balance": {
                        "value": 23180.583333333332
                    }
                },
                {
                    "key": "MA",
                    "doc_count": 24,
                    "average_balance": {
                        "value": 29600.333333333332
                    }
                },
                {
                    "key": "ND",
                    "doc_count": 24,
                    "average_balance": {
                        "value": 26577.333333333332
                    }
                }
            ]
        }
    }
}
```

你可以通过指定 `terms` 聚合内的顺序来使用嵌套聚合的结果进行排序，而不是按计数进行排序：

```PHP
$params = [
    'index' => 'bank',
    'body' => [
        'size' => 0,
        'aggs' => [
            'group_by_state' => [
                'terms' => [
                    'field' => 'state.keyword',
                    'order' => [
                        'average_balance' => 'desc',
                    ],
                ],
                'aggs' => [
                    'average_balance' => [
                        'avg' => [
                            'field' => 'balance',
                        ],
                    ],
                ],
            ],
        ],
    ],
];

$response = $client->search($params);
```

结果：

```
{
    "took": 9,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1000,
            "relation": "eq"
        },
        "max_score": null,
        "hits": []
    },
    "aggregations": {
        "group_by_state": {
            "doc_count_error_upper_bound": -1,
            "sum_other_doc_count": 827,
            "buckets": [
                {
                    "key": "CO",
                    "doc_count": 14,
                    "average_balance": {
                        "value": 32460.35714285714
                    }
                },
                {
                    "key": "NE",
                    "doc_count": 16,
                    "average_balance": {
                        "value": 32041.5625
                    }
                },
                {
                    "key": "AZ",
                    "doc_count": 14,
                    "average_balance": {
                        "value": 31634.785714285714
                    }
                },
                {
                    "key": "MT",
                    "doc_count": 17,
                    "average_balance": {
                        "value": 31147.41176470588
                    }
                },
                {
                    "key": "VA",
                    "doc_count": 16,
                    "average_balance": {
                        "value": 30600.0625
                    }
                },
                {
                    "key": "GA",
                    "doc_count": 19,
                    "average_balance": {
                        "value": 30089
                    }
                },
                {
                    "key": "MA",
                    "doc_count": 24,
                    "average_balance": {
                        "value": 29600.333333333332
                    }
                },
                {
                    "key": "IL",
                    "doc_count": 22,
                    "average_balance": {
                        "value": 29489.727272727272
                    }
                },
                {
                    "key": "NM",
                    "doc_count": 14,
                    "average_balance": {
                        "value": 28792.64285714286
                    }
                },
                {
                    "key": "LA",
                    "doc_count": 17,
                    "average_balance": {
                        "value": 28791.823529411766
                    }
                }
            ]
        }
    }
}
```

除了这些基本的聚合外，Elasticsearch 还提供了专门的聚合，用于在多个字段上操作并分析特定类型的数据，例如日期，IP 地址和地理数据。您还可以将单个聚合的结果发送到聚合管道中，以便进行进一步分析。

聚合提供的核心分析功能可启用高级功能，例如使用机器学习来检测异常。
