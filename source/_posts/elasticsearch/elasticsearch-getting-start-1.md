---
title: Elasticsearch 入门（一）- 索引文档
date: 2020-06-29 11:47:00
tags: [ElasticSearch]
---

### 索引单个文档

我们可以使用一个 `PUT` 请求来索引单个文档：

```
PUT /customer/_doc/1
{
    "name": "John Doe"
}
```

* 使用 `PUT` 方法
* `customer`: 索引名称
* `1`: 唯一的文档 ID
* body: 文档内容，可以包含一个或多个键值对

如果索引不存在，则 Elasticsearch 会新建一个，然后存储该文档。

因为这是一个新的文档，所以响应结果显示文档的版本为 1：

```
{
  "_index" : "customer",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 26,
  "_primary_term" : 4
}
```

### 获取单个文档

我们可以通过一个 `GET` 请求来获取刚刚保存进索引的文档：

```
GET /customer/_doc/1
```

这里的 `1` 是文档唯一 ID。

响应结果如下，`_source` 是原始文档内容：

```
{
	"_index": "customer",
	"_type": "_doc",
	"_id": "1",
	"_version": 2,
	"_seq_no": 1,
	"_primary_term": 1,
	"found": true,
	"_source": {
		"h": [
			"health",
			"index",
			"docs.count"
		]
	}
}
```


### 使用 bulk API 批量索引文档

如果你有很多要索引的文档，则可以使用 [bulk API](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-bulk.html) 批量提交。使用批量处理批处理文档操作比单独提交请求要快得多，因为它可以最大程度地减少网络往返次数。


### 导入测试数据

[https://www.elastic.co/guide/en/elasticsearch/reference/master/getting-started-index.html#getting-started-index](https://www.elastic.co/guide/en/elasticsearch/reference/master/getting-started-index.html#getting-started-index)
