---
title: laravel jenssegers mongodb 遍历大数据量 collection 优化
date: 2020-02-24 15:06:00
tags: [MongoDB, Laravel, PHP]
---

## 环境: 

* `Lumen 5.5.*`

* `PHP 7.1.14`

* `ext-mongodb 1.5.2`


## 背景

线上 Mongo 的某个 Collection 数据量过大(400w+)，导致某些搜索操作很慢，所以考虑对其进行拆分，拆分之后可能平均每个 Collection 2w 左右数据。

所以写了一个脚本来遍历该 Collection，把拿到的数据根据其中一个字段拆分到不同的 Collection 中。

跑这个脚本的时候发现，开始的时候挺快的，然后跑着跑着就越来越慢了。


## 代码

大致做法，每次获取 1000 条数据（避免占用过大内存），处理完这 1000 条数据之后，再获取 1000 条，直到处理完所有数据。

### 第一版

```
app(xx::class)
    ->withTrashed()
    ->chunk(1000, function($xxs) {
    });
```

**总运行时间（150w 数据）：> 1h**

这个脚本对有 150w 数据的 Collection 的时候就遇到了严重的性能问题，跑到一半的时候，脚本报告跑完这个脚本可能总共要 1 个小时。

因此，在它跑的过程中，Google 了一下为什么这么慢，并且做了一些测试，发现 `jenssegers` 使用的方法是 [aggregate](https://docs.mongodb.com/manual/aggregation/)，另外发现 `jenssegers` 上有个 issue 质疑为什么用 `aggregate`（[Why use aggregation to paginate? ](https://github.com/jenssegers/laravel-mongodb/issues/1056)）。有一位仁兄提到 "mongodb is used for performance mainly, but using aggregate instead of find is killing the purpose."。

显然，大家对此都不太满意。我也是。所以有了第二版。


### 第二版

这版写法来自上面提到的那个 issue（（[Why use aggregation to paginate? ](https://github.com/jenssegers/laravel-mongodb/issues/1056)））。

既然这个人说 `db.{dbname}.find().sort({"_id":-1}).skip(2).limit(2)` 这种写法更优，那必须得尝试一下：

```
app(xx::class)
    ->withTrashed()
    ->skip($page * $pageSize)
    ->limit($pageSize)
    ->get();
```

**总运行时间（150w 数据）：0.5h**

总的来说，快了很多很多，但是这种解决方法也还是存在同样的一个问题，就是越到后面就越慢。如果数据量更多的话，没有办法确保执行时间在可控范围之内。


### 第三版

我们知道，对于一些数据库，提供了游标这种东西，可能大部分人没有去用过。出现上面这种状况，我想大概是因为，分页的操作实际上是，一条条数据数过去，直到某一条数据吧。

为了证实这个想法，百度了一下 MySQL 版本的，大概底层实现其实就是一条条数过去的：

```
mysql> EXPLAIN SELECT * FROM users ORDER BY id DESC LIMIT 10000, 20\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: users
   partitions: NULL
         type: index
possible_keys: NULL
          key: PRIMARY
      key_len: 4
          ref: NULL
         rows: 10020  # 扫描了 10020 行
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.03 sec)
```

`LIMIT 10000, 20` 意味着 MySQL 已经读取了 10020 条数据，并且丢弃了前 10000 行，然后返回接下来的 20 行。

参考链接: [Efficient Pagination Using MySQL](https://www.slideshare.net/Eweaver/efficient-pagination-using-mysql)

> 也就是说，我们遍历的时候越到后面的数据，每次获取就需要丢弃越来越多的数据，简单来说，越到后面，吃力不讨好的操作越来越多，所以也就越来越慢。

* **有没有办法让它下次不再重新找？**

所以我们在这种场景下需要解决的问题是，让 mongo 不要每次都从头找，你已经到这了，不用回头再来。想到这，想起了有 `cursor` 这个东西，就是游标。MySQL 里面也有，它可以在查询的过程中给你一个游标，然后再自己一条条地拿数据。


使用 `cursor` 版本的代码：

```
// 获取游标
/** @var \MongoDB\Driver\Cursor $cursor */
$cursor = DB::connection('<connection_name>')
    ->collection('<collection_name>')->raw(function ($collection) {
        return $collection->find();
    });

$count = 0;
$data = [];
/** @var \MongoDB\Model\BSONDocument $bsonDocument */
foreach ($cursor as $bsonDocument) {
    $json = \MongoDB\BSON\toJSON(\MongoDB\BSON\fromPHP($bsonDocument));
    $model = app(xx::class);
    $attributes = json_decode($json, true);

    foreach ($attributes as $attribute => $value) {
        $model->setAttribute($attribute, $bsonDocument[$attribute]);
    }

    $data[] = $model;
    $count++;
    if ($count == 1000) {
        $this->insertDocuments($data);
        $data = [];
        $count = 0;
    }
}

if ($count > 0) {
    $this->insertDocuments($data);
}
```

操作起来繁琐了很多，我们需要先获取 `cursor`，然后在循环中手动控制每次处理的量。

**总运行时间（150w 数据）：9min**

总的运行时间少了很多，而且不存在处理到后面的时候越来越慢，匀速处理，这个方案目前来看算是最优的了。


## 总结

* 使用 cursor 遍历大数据量 collection 的时候可以避免处理到后面的数据越来越慢。
