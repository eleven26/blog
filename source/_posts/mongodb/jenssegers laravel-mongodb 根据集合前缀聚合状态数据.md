---
title: jenssegers laravel-mongodb 根据集合前缀聚合状态数据
date: 2021-06-08 09:12:00
tags: [MongoDB, Laravel, PHP]
---

有时候需要根据需要对 MongoDB 的集合进行分集合存储（类似 MySQL 的分表），这样一来 db 下就产生了很多相同前缀的集合，这样一来，我们想知道这一批集合的一些关键状态信息的时候就比较不便，比如想知道某一个前缀所有集合的总大小等等。

一个可行的方法是，通过找到一个模式前缀的所有集合，然后逐个调用 MongoDB 的 [collStats](https://docs.mongodb.com/manual/reference/command/collStats/) 命令来获取单个集合的状态，然后做一个聚合，下面是基于 [jenssegers mongodb](https://github.com/jenssegers/laravel-mongodb) 的一个实现：


## 统计 product 库下 xx_ 前缀的所有集合的大小等

输出的字段说明：

* `collection_count`: `xx_` 前缀的集合总数量
* `total_document_count`: `xx_` 前缀文档总数量
* `total_size`: `xx_` 前缀集合总大小
* `avg_obj_size`: `xx_` 前缀集合平均单个文档的大小
* `total_storage_size`: `xx_` 前缀集合占的总存储大小
* `total_index_size`: `xx_` 前缀集合索引总大小
* `total_bytes_in_cache`: `xx_` 前缀集合数据当前占用的缓存大小

```PHP
$patterns = [
    'product.xx_'
];

function format($bytes): string
{
    if ($bytes < 1024) {
        return "{$bytes} bytes";
    }

    if ($bytes < 1024 * 1024) {
        $bytes = round($bytes / 1024, 2);

        return "{$bytes} Kb";
    }

    if ($bytes < 1024 * 1024 * 1024) {
        $bytes = round($bytes / 1024 / 1024, 2);

        return "{$bytes} Mb";
    }

    $bytes = round($bytes / 1024 / 1024 / 1024, 2);

    return "{$bytes} Gb";
}

class CollectionStats
{
    private $stats;

    public function __construct(array $stats)
    {
        $this->stats = $stats;
    }

    public function toArray(): array
    {
        return [
            'collection'     => $this->stats['ns'],
            'size'           => format($this->stats['size']),
            'count'          => $this->stats['count'],
            'avgObjSize'     => format($this->stats['avgObjSize']),
            'storageSize'    => format($this->stats['storageSize']),
            'totalIndexSize' => format($this->stats['totalIndexSize']),

            'raw' => [
                'count'          => $this->stats['count'],
                'size'           => $this->stats['size'],
                'avgObjSize'     => $this->stats['avgObjSize'],
                'storageSize'    => $this->stats['storageSize'],
                'totalIndexSize' => $this->stats['totalIndexSize'],
                'bytes_in_cache' => $this->stats['wiredTiger']['cache']['bytes currently in the cache'],
            ],
        ];
    }
}

foreach ($patterns as $pattern) {
    // $connectionName 连接名
    // $collectionPrefix Collection 前缀
    [$connectionName, $collectionPrefix] = explode('.', $pattern);

    dump(compact('connectionName', 'collectionPrefix'));

    $collections = get_collection_names($connectionName);
    $collections = collect($collections)
        ->filter(function ($collection) use ($collectionPrefix) {
            return Str::startsWith($collection, $collectionPrefix);
        })
        ->values()
        ->toArray();

    // 查看 Collection 的状态：
    // https://docs.mongodb.com/manual/reference/command/
    // https://docs.mongodb.com/manual/reference/command/collStats/#mongodb-dbcommand-dbcmd.collStats

    $result = [];

    foreach ($collections as $collection) {
        $cursor = DB::connection($connectionName)->getMongoDB()->command(['collStats' => $collection]);
        $res     = json_decode(json_encode($cursor->toArray()[0]), true);
        $reply   = json_encode($res, JSON_UNESCAPED_UNICODE | JSON_PRETTY_PRINT);

        $result[] = (new CollectionStats($res))->toArray();
    }

    $results = collect($result);

    dump('collection_count: ' . $results->count());
    dump('total_document_count: ' . $results->sum('raw.count'));
    dump('total_size: ' . format($results->sum('raw.size')));
    dump('avg_obj_size: ' . format($results->avg('raw.avgObjSize')));
    dump('total_storage_size: ' . format($results->sum('raw.storageSize')));
    dump('total_index_size: ' . format($results->sum('raw.totalIndexSize')));
    dump('total_bytes_in_cache: ' . format($results->sum('raw.bytes_in_cache')));
    echo PHP_EOL;
}
```

输出：

```console
array:2 [
  "connectionName" => "product"
  "collectionPrefix" => "xx_"
]
"collection_count: 222"
"total_document_count: 1664601"
"total_size: 2.24 Gb"
"avg_obj_size: 1.4 Kb"
"total_storage_size: 996.66 Mb"
"total_index_size: 156.34 Mb"
"total_bytes_in_cache: 3.17 Gb"
```

## 其他

这个例子是基于 Laravel 的一个库来实现的，不过只是简单地调用了一个 `command` 方法，其他语言利用 MongoDB 提供的库也比较容易可以实现。
