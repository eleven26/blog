---
title: Elasticsearch 嵌套对象之嵌套类型
date: 2020-07-02 09:17:04
tags: [ElasticSearch]
---

nested 类型是一种特殊的对象 object 数据类型，允许对象数组彼此独立地进行索引和查询。

## 对象数组如何扁平化

内部对象 object 字段的数组不能像我们期望的那样工作。Lucene 没有内部对象的概念，所以 Elasticsearch 将对象层次结构扁平化为一个字段名称和值简单列表。例如，以下文件：

```
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -H 'Content-type: application/json' -d '
{
    "group": "fans",
    "user": [
        {
            "first": "John",
            "last": "Smith"
        },
        {
            "first": "Alice",
            "last": "White"
        }
    ]
}
'
```

说明

user 字段被动态的添加为 object 类型的字段。

在内部其转换成一个看起来像下面这样的文档：

```
{
    "group": "fans",
    "user.first": ["alice", "john"],
    "user.last": ["smith", "white"]
}
```

`user.first` 和 `user.last` 字段被扁平化为多值字段，并且 alice 和 white 之间的关联已经丢失。本文档将错误地匹配 `user.first` 为 alice 和 `user.last` 为 smith 的查询：

```
curl -XGET 'localhost:9200/my_index/_search?pretty' -H 'Content-Type: application/json' -d '
{
    "query": {
        "bool": {
            "must": [
                {
                    "match": {
                        "user.first": "Alice"
                    }
                },
                {
                    "match": {
                        "user.last": "Smith"
                    }
                }
            ]
        }
    }
}
'
```


## 对对象数组使用嵌套字段

如果需要索引对象数组并维护数组中每个对象的独立性，则应使用 nested 数据类型而不是 object 数据类型。在内部，嵌套对象将数组中的每个对象作为单独的隐藏文档进行索引，这意味着每个嵌套对象都可以使用嵌套查询 nested query 独立于其他对象进行查询：

```
curl -XPUT 'localhost:9200/my_index?pretty' -H 'Content-Type: application/json' -d '
{
    "mappings": {
        "my_type": {
            "properties": {
                "user": {
                    "type": "nested"
                }
            }
        }
    }
}
'
```

```
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -H 'Content-Type: application/json' -d '
{
    "group": "fans",
    "user": [
        {
            "first": "John",
            "last": "Smith"
        },
        {
            "first": "Alice",
            "last": "White"
        }
    ]
}
'
```

说明：

user 字段映射为 nested 类型，而不是默认的 object 类型。

```
curl -XGET 'localhost:9200/my_index/_search?pretty' -H 'Content-Type: application/json' -d '
{
    "query": {
        "nested": {
            "path": "user",
            "query": {
                "bool": {
                    "must": [
                        {
                            "match": { "user.first": "Alice"}
                        },
                        {
                            "match": { "user.last": "Smith" }
                        }
                    ]
                }
            }
        }
    }
}
'
```

说明：

此查询得不到匹配，是因为 Alice 和 Smith 不在同一个嵌套对象中。

```
curl -XGET 'localhost:9200/my_index/_search?pretty' -H 'Content-Type: application/json' -d '
{
    "query": {
        "nested": {
            "path": "user",
            "query": {
                "bool": {
                    "must": [
                      { "match": { "user.first": "Alice" } },
                      { "match": { "user.last": "White" } }
                    ]
                }
            },
            "inner_hits": {
                "highlight": {
                    "fields": {
                        "user.first": {}
                    }
                }
            }
        }
    }
}
'
```

说明：

此查询得到匹配，是因为 Alice 和 White 位于同一个嵌套对象中。

inner_hits 允许我们突出显示匹配的嵌套文档。

嵌套文档可以：

* 使用 nested 查询进行查询
* 使用 nested 和 reverse_nested 聚合进行分析
* 使用 nested 排序进行排序
* 使用 nested_inner_hits 进行检索与突出显示


## 嵌套字段参数

嵌套字段接受以下参数：

* dynamic：是否将新属性动态添加到现有的嵌套对象。共有 true（默认），false 和 strict 三种参数。
* include_in_all：(_all 字段已经废弃了)
* properties：嵌套对象中的字段，可以是任何数据类型，包括嵌套。新的属性可能会添加到现有的嵌套对象。

备注：

类型映射（type mapping）、对象字段和嵌套字段包含的子字段，称之为属性 properties。这些属性可以为任意数据类型，包括 object 和 nested。属性可以通过以下方式加入：

* 当在创建索引时显式定义它们
* 当使用 PUT mapping API 添加或更新映射类型时显式地定义它们
* 当索引包含新字段的文档时动态的加入

重要：

由于嵌套文档作为单独的文档进行索引，因此只能在 nested 查询，nested/reverse_nested 聚合或者 nested_inner_hits 的范围内进行访问。


## 限制嵌套字段的个数

索引一个拥有 100 个嵌套字段的文档，相当于索引了 101 个文档，因为每一个嵌套文档都被索引为一个独立的文档。为了防止不明确的映射，每个索引可以定义的嵌套字段的数量已被限制为 50 个。

多嵌套查询方式：

```
{
    "query": {
        "bool": {
            "must": [
                {
                    "nested": {
                        "path": [
                            "ajxx"
                        ],
                        "query": {
                            "bool": {
                                "must": [
                                    {
                                        "match": {
                                            "ajxx.ajzt": "破案"
                                        }
                                    },
                                    {
                                        "range": {
                                            "ajxx.sasj": {
                                                "gte": "2017-01-01 12:10:10",
                                                "lte": "2017-01-02 12:10-40"
                                            }
                                        }
                                    }
                                ],
                                "should": [
                                    {
                                        "query_string": {
                                            "query": "20170316盗窃案"
                                        }
                                    }
                                ]
                            }
                        }
                    }
                }
            ]
        }
    }
}
```

查询字段名称的模糊匹配编辑
字段名称可以用模糊匹配的方式给出：任何与模糊式正则匹配的字段都会被包括在搜索条件中。

```
{
    "multi_match": {
        "query": "结果",
        "fields": ["*", "*_title"]
    }
}
```

当这些子字段出现数值类型的时候，就会报异常了，解决方法是加入 lenient 字段。

```
{
    "type": "parse_exception",
    "reason": "failed to parse date field [XXX] with format [yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis]"
}

{
    "multi_match": {
        "query": "结果",
        "lenient": "true",
        "fields": ["*"]
    }
}
```


利用 multi_match 嵌套全文检索

```
"include_in_parent": true,
"include_in_root": true,
{
    "query": {
        "bool": {
            "must": [
                {
                    "match": {
                        "ajztmc": "立案"
                    }
                },
                {
                    "match": {
                        "zjasl": "刑事"
                    }
                },
                {
                    "range": {
                        "lasj": {
                            "gte": "2015-01-01 12:10:10",
                            "lte": "2016-01-01 12:10:40"
                        }
                    }
                },
                {
                    "nested": {
                        "path": [
                            "rqxxx"
                        ],
                        "query": {
                            "bool": {
                                "must": [
                                    {
                                        "match": {
                                            "rqxx.baqmc": "办案区名称"
                                        }
                                    }
                                ]
                            }
                        }
                    }
                },
                {
                    "nested": {
                        "path": [
                            "saryxx"
                        ],
                        "query": {
                            "bool": {
                                "must": [
                                    {
                                        "match": {
                                            "abc": "嫌疑人"
                                        }
                                    },
                                    {
                                        "match": {
                                            "def": "女"
                                        }
                                    }
                                ]
                            }
                        }
                    }
                },
                {
                    "nested": {
                        "path": [
                            "wp"
                        ],
                        "query": {
                            "bool": {
                                "must": [
                                    {
                                        "match": {
                                            "wp.wpzlnc": "赃物"
                                        }
                                    },
                                    {
                                        "match": {
                                            "wp.wpztmc": "物品入库"
                                        }
                                    }
                                ]
                            }
                        }
                    }
                },
                {
                    "multi_match": {
                        "query": "男",
                        "lenient",
                        "fields": [
                            "*"
                        ]
                    }
                }
            ]
        }
    },
    "from": 0,
    "size": 100,
    "sort": {
        "zxxgsj": {
            "order": "desc"
        }
    }
}
```

