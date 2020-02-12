---
title: jenssegers mongodb 记录查询语句
date: 2020-02-12 18:38:00
tags: [Laravel, PHP]
---

## 机制：

`jenssegers mongodb` 是否记录查询语句依赖于 `\Illuminate\Database\Connection` 里面的 `loggingQueries` 属性是否为 true，
而这个属性可以使用 `DB::connection('xx')->enableQueryLog()` 来设置，我们调用了这个方法之后，该扩展包底层会 fire 一个
 `\Illuminate\Database\Events\QueryExecuted` 事件，所以如果我们想把查询语句记录到日志文件，可以考虑，监听这个事件，然后在
 回调里面写入日志。


`\Jenssegers\Mongodb\Collection`

```
/**
 * Handle dynamic method calls.
 *
 * @param  string $method
 * @param  array $parameters
 * @return mixed
 */
public function __call($method, $parameters)
{
    $start = microtime(true);
    $result = call_user_func_array([$this->collection, $method], $parameters);

    if ($this->connection->logging()) {
        // Once we have run the query we will calculate the time that it took to run and
        // then log the query, bindings, and execution time so we will report them on
        // the event that the developer needs them. We'll log time in milliseconds.
        $time = $this->connection->getElapsedTime($start);

        $query = [];

        // Convert the query parameters to a json string.
        array_walk_recursive($parameters, function (&$item, $key) {
            if ($item instanceof ObjectID) {
                $item = (string) $item;
            }
        });

        // Convert the query parameters to a json string.
        foreach ($parameters as $parameter) {
            try {
                $query[] = json_encode($parameter);
            } catch (Exception $e) {
                $query[] = '{...}';
            }
        }

        $queryString = $this->collection->getCollectionName() . '.' . $method . '(' . implode(',', $query) . ')';

        $this->connection->logQuery($queryString, [], $time);
    }

    return $result;
}
```

## 启用查询 log

```
DB::connection('yy')->enableQueryLog();
$model = app(xx::class);
$model->limit(10)->get();
```

## 监听事件

在哪里写这个可以参考官方文档，这里只展示具体代码：

```
\DB::listen(function (QueryExecuted $sql) use ($logSql) {
    \Log::info('time: ' . $sql->time . '  ' . $sql->sql);
});
```

