---
title: Laravel 如何更快地遍历全表数据
date: 2020-05-13 10:00:00
tags: [Laravel, PHP]
---

有时候我们可能需要查询某一个表的全部数据做一些处理，这个时候有一个可行的方法就是直接调用模型的 `get()` 方法，又或者调用 `chunk()` 方法。

但是这两种方案在处理大表的时候都不好，首先是 `get()`，会导致 PHP 占用内存过大，而 `chunk()` 方法实际上就是一个分页的封装，最终的查询语句是 `LIMIT offset, count;`。

`chunk()` 也是个人之前一直使用的方法，但是在表越来越大之后，发现有比较严重的性能问题，越到后面的页查询就越慢。


## 为什么 MySQL 分页慢？

我们可以 explain 一下分页 sql，我们会发现扫描行数等于 `limit offset, count` 里面的 `offset`，这和 MySQL 的分页机制有关：

MySQL 在执行 `limit offset, count` 语句的时候，需要把第一条数据到 `offset` 的那一条数据扫描一遍。

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


## 使用 chunk 有什么问题？

现在我们知道了，MySQL 在分页的时候是从第一条数到 `offset` 的，也就是说，我们的 `offset` 越往后，需要扫描的行就越多。

> 在我们需要遍历全表数据的这种场景下，MySQL 就需要不断地扫描之前扫描过的数据，这样会导致重复扫描非常多。

我们都知道扫描一遍只需要 O(n) 的时间，但是由于 MySQL 的这种机制加上 chunk，会直接导致时间复杂度增加为 O(n²) ，在我们数据量越多的时候，速度下降得就越快。


## 解决方法


### 1. 记录上一次的最大 id（推荐使用）

> 在 MySQL 中的 InnoDB 引擎，主键索引字段是一个聚簇索引，存在 B+ 树的叶子节点层，是有序的。

我们可以利用这个特点，将上一次的最大 id （主键）记录下来，假设是 lastId，然后下一次查询的时候，加上 `where id > lastId`，这个时候我们的 limit 语句也要改一下，改成 `limit count`，就可以了，因为我们告诉了 MySQL offset 是什么。这样 MySQL 就不用做一些重复的扫描操作了。

代码：

```PHP
// 我们需要按 id 顺序去遍历
$builder = app(\Modules\Product\Models\Sku::class)->orderBy('id');

$lastId = 0;
while (true) {
    /** @var \Illuminate\Database\Eloquent\Collection $skus */
    $skus = app(\Modules\Product\Models\Sku::class)
        ->where('id', '>', $lastId)
        ->orderBy('id')
        ->limit(100)
        ->get();

    if ($skus->count() > 0) {
        $lastId = $skus->max('id');
    }

    // 最后一页了
    if ($skus->count() < 100) {
        break;
    }
}
```

测试结果：在 33w 表的情况下，`chunk()` 需要 390s，而按上述方法只需要 22s。


### 2. 利用 MYSQL_ATTR_USE_BUFFERED_QUERY

在 PDO 里面有一个常量 MYSQL_ATTR_USE_BUFFERED_QUERY，是用来告诉 MySQL 是否使用查询缓存的。

Laravel 里面提供了一个 `cursor()` 方法，但是实际查询的时候是先获取所有结果再往下处理的，并不是预期那样获取一条之后返回。可参考 [Using Cursor on large number of results causing memory issues](https://github.com/laravel/framework/issues/14919)。这个方法想要做的事情的确和我们的想法契合，但是由于 PDO 的 MYSQL_ATTR_USE_BUFFERED_QUERY 默认值为 true。所以导致实际表现并不是我们想要的。

但是 Laravel 也提供了方法让我们去手动设置这个属性：

```
$builder = app(\Modules\Product\Models\Sku::class);
// 获取底层 pdo 对象，然后设置 \PDO::MYSQL_ATTR_USE_BUFFERED_QUERY 为 false
$builder->getConnection()
    ->getPdo()
    ->setAttribute(\PDO::MYSQL_ATTR_USE_BUFFERED_QUERY, false);

foreach ($builder->cursor() as $item) {
}
```

这种解决方法有一定的问题，可参考上面提到的那个 laravel 的 issue。


## 总结

Laravel 扫描全表的时候可以记录上次 `get()` 的最大 id，下次从这个 id 起扫描，又或者利用 pdo 的 `MYSQL_ATTR_USE_BUFFERED_QUERY` 属性来单条获取。

