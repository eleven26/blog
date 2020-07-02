---
title: Elasticsearch 查询 - match 和 term
date: 2020-07-02 09:17:02
tags: [ElasticSearch]
---

elasticsearch 中的查询请求有两种方式，一种是简易版的查询，另外一种是使用 JSON 完整的请求体，叫做结构化查询（DSL）。由于 DSL 查询更为直观也更为简易，所以大都使用这种方式。DSL 查询是 POST 过去一个 json，由于 post 的请求是 json 格式的，所以存在很多灵活性，也有很多形式。这里有一个地方注意的是官方文档里面给的例子的 json 结构只是一部分，并不是可以直接复制粘贴进去使用的。一般要在外面加个 query 为 key 的结构。


## match

最简单的一个 match 例子：

查询和 “我的宝马多少马力” 这个查询语句匹配的文档。

```
{
    "query": {
        "match": {
            "content": {
                "query": "我的宝马多少马力"
            }
        }
    }
}
```

上面的查询匹配就会进行分词，比如 “宝马多少马力” 会被分词为 “宝马 多少 马力”，所有有关 “宝马 多少 马力”，那么所有包含这三个词中的一个或多个的文档就会被搜索出来。并且根据 lucene 的评分机制（TF/IDF）来进行评分。


## match_phrase

比如上面一个例子，一个文档 “我的保时捷马力也不错” 也会被搜索出来，那么想要精确匹配所有同时包含 “宝马 多少 马力” 的文档怎么做？就要使用 match_phrase 了。

```
{
    "query": {
        "match_phrase": {
            "content": {
                "query": "我的宝马多少马力"
            }
        }
    }
}
```

完全匹配可能比较严，我们会希望有个可调节因子，少匹配一个也满足，那就需要使用到 slop。

```
{
    "query": {
        "match_phrase": {
            "content": {
                "query": "我的宝马多少马力",
                "slop": 1
            }
        }
    }
}
```

## multi_match

如果我们希望两个字段进行匹配，其中一个字段有这个文档就满足的话，使用 multi_match

```
{
    "query": {
        "multi_match": {
            "query": "我的宝马多少马力",
            "fields": ["title", "content"]
        }
    }
}
```

但是 multi_match 就涉及到匹配评分的问题了。


### 我们希望完全匹配的文档占的评分比较高，则需要使用 best_fields

```
{
    "query": {
        "multi_match": {
            "query": "我的宝马发动机多少",
            "type": "best_fields",
            "fields": [
                "tag",
                "content"
            ],
            "tie_breaker": 0.3
        }
    }
}
```

意思就是完全匹配 “宝马 发动机” 的文档评分会比较靠前，如果只匹配宝马的文档评分乘以 0.3 的系数。


### 我们希望越多字段匹配的文档评分越高，就要使用 most_fields

```
{
    "query": {
        "multi_match": {
            "query": "我的宝马发动机多少",
            "type": "most_fields",
            "fields": [
                "tag",
                "content"
            ]
        }
    }
}
```

### 我们希望这个词条的分词词汇是分配到不同字段中的，那么就使用 cross_fields

```
{
    "query": {
        "multi_match": {
            "query": "我的宝马发动机多少",
            "type": "cross_fields",
            "fields": [
                "tag",
                "content"
            ]
        }
    }
}
```


## term

term 是代表完全匹配，即不进行分词器分析，文档中必须包含整个搜索的词汇

```
{
    "query": {
        "term": {
            "content": "汽车保养"
        }
    }
}
```

查询出的所有文档都包含 “汽车保养” 这个词组的词汇。

使用 term 要确定的是这个字段是否 “被分析（analyzed）”，默认的字符串是被分析的。

拿官网上的例子举例：

mapping 是这样的：

```
PUT my_index
{
    "mappings": {
        "my_type": {
            "properties": {
                "full_text": {
                    "type": "string"
                },
                "exact_value": {
                    "type": "string",
                    "index": "not_analyzed"
                }
            }
        }
    }
}

PUT my_index/my_type/1
{
    "full_text": "Quick Foxes!",
    "exact_value": "Quick Foxes!"
}
```

其中的 full_text 是被分析过的，所以 full_text 的索引中存的就是 [quick, foxes]，而 extra_value 中存的是 [Quick Foxes!]。

那下面的几个请求：

```
GET my_index/my_type/_search
{
    "query": {
        "term": {
            "exact_value": "Quick Foxes!"
        }
    }
}
```

能请求得的数据，因为完全匹配。

```
GET my_index/my_type/_search
{
    "query": {
        "term": {
            "full_text": "Quick Foxes!"
        }
    }
}
```

请求不出数据，因为 full_text 分词后的结果中没有 [Quick Foxes!] 这个分词。


## bool 联合查询：must，should，must_not

如果我们想要请求 “content 中带宝马，但是 tag 中不带宝马” 类似这样的需求，就需要用到 bool 联合查询。联合查询就会使用到 must，should，must_not 三种关键词。

这三个可以这么理解：

* must：文档必须完全匹配条件
* should：should 下面会带有一个以上的条件，至少满足一个条件，这个文档就符合 should
* must_not：文档必须不匹配条件

比如上面那个需求：

```
{
    "query": {
        "bool": {
            "must": {
                "term": {
                    "content": "宝马"
                }
            },
            "must_not": {
                "term": {
                    "tags": "宝马"
                }
            }
        }
    }
}
```
