---
title: Laravel 5.5 关联 MongoDB whereHas 查询不到数据的问题
date: 2021-06-08 09:40:00
tags: [Laravel, PHP]
---

具体情况大概是这样的，我们有一个 MySQL 的模型，有一个 Mongo 的模型，然后在 MySQL 的模型中 hasOne Mongo 的模型，结果：

MySQL with 的时候可以查询出数据，但是使用 whereHas 的时候查询不到数据了。


## 环境

* `laravel/lumen-framework`: 5.5.2
* `jenssegers/mongodb`: 3.3.2


## 重现

```PHP
class A extends \Illuminate\Database\Eloquent\Model
{
    // 可以让我们关联 MongoDB 的一个 trait
    use \Jenssegers\Mongodb\Eloquent\HybridRelations;

    public function b()
    {
        return $this->hasOne(B::class, '_id', 'b_id');
    }
}

class B extends \Jenssegers\Mongodb\Eloquent\Model
{
    
}
```

Laravel 的关联查询如下：

```PHP
$a = A::query()->with('b')->whereKey(10)->first();
// $a 可以查询出来
dump($a);

$a = A::query()->whereHas('b')->whereKey(10)->first();
// $a 为 null
dump($a);
```

正常情况下，其实第二个查询我们也应该查询出来才对，可是我们查不到，然后尝试打印一下查询：

```
[2021-06-08 09:54:30] lumen.INFO: time: 75.18 select * from `as` where `as`.`id` = 10 limit 1  
[2021-06-08 09:54:30] lumen.INFO: time: 48.25,  connection: mongo_core,  db.bs.find({"_id":{"$in":["60becd2306bce821e55ffce3"]}})  
[2021-06-08 09:54:30] lumen.INFO: time: 2.52,  connection: mongo_core,  db.bs.find({"projection":{"_id":true}})  
[2021-06-08 09:54:30] lumen.INFO: time: 18.81 select * from `as` where `id` in ("60becd2306bce821e55ffce3") and `as`.`id` = 10 limit 1  
```

我们发现最后一条语句并不是如我们期待的那样，`where b_id in` 而是 `where id in`，这是一个不正常的行为，因为根据我们的定义，关联 A 模型的字段应该是 `b_id` 而不是 `id`。

## 原因

因为想找到查询使用 `id` 而不是 `b_id`，对关联部分的代码 debug 了一下，最后发现，位于 `\Jenssegers\Mongodb\Helpers\QueriesRelationships` 的方法 `getRelatedConstraintKey` 返回了一个错误的关联键：

[QueriesRelationships.php v3.3.2](https://github.com/jenssegers/laravel-mongodb/blob/v3.3.2/src/Jenssegers/Mongodb/Helpers/QueriesRelationships.php)

```PHP
/**
 * Returns key we are constraining this parent model's query with
 * @param $relation
 * @return string
 * @throws \Exception
 */
protected function getRelatedConstraintKey($relation)
{
    if ($relation instanceof HasOneOrMany) {
        // 这里返回了关联模型的主键（id），但是实际上应该返回的是关联里面指定的那个键（b_id）
        return $this->model->getKeyName();
    }

    if ($relation instanceof BelongsTo) {
        return $relation->getForeignKey();
    }

    throw new \Exception(class_basename($relation) . ' Is Not supported for hybrid query constraints!');
}
```

感觉这是一个 bug，然后去新版本的库里面看了一下，发现新版本里面已经修改过来了：

[QueriesRelationships.php v3.7.3](https://github.com/jenssegers/laravel-mongodb/blob/v3.7.3/src/Jenssegers/Mongodb/Helpers/QueriesRelationships.php)

```PHP
/**
 * Returns key we are constraining this parent model's query with
 * @param Relation $relation
 * @return string
 * @throws Exception
 */
protected function getRelatedConstraintKey(Relation $relation)
{
    if ($relation instanceof HasOneOrMany) {
        // 返回了正确的关联键
        return $relation->getLocalKeyName();
    }

    if ($relation instanceof BelongsTo) {
        return $relation->getForeignKeyName();
    }

    if ($relation instanceof BelongsToMany && !$this->isAcrossConnections($relation)) {
        return $this->model->getKeyName();
    }

    throw new Exception(class_basename($relation) . ' is not supported for hybrid query constraints.');
}
```

## 结论

由于修复的版本与当前使用的版本相差太多，而且依赖于 Laravel 的版本，所以无法升级其版本，只有换一种写法了。（whereHas 拆分出来，先查询到关联数据的 _id，然后作为 whereIn 条件传递）。
