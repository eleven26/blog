---
title: Laravel 合并模型集合的注意事项
date: 2020-02-15 18:36:00
tags: [Laravel, PHP]
---


## 源码

\Illuminate\Database\Eloquent\Collection::merge

```
/**
 * Merge the collection with the given items.
 *
 * @param  \ArrayAccess|array  $items
 * @return static
 */
public function merge($items)
{
    $dictionary = $this->getDictionary();

    foreach ($items as $item) {
        $dictionary[$item->getKey()] = $item;
    }

    return new static(array_values($dictionary));
}
```

模型集合合并的时候，如果存在相同主键值的模型，那么合并的结果中，只会有相同主键值模型的最后一个。相同主键值的模型会被忽略掉。


## 不同模型相同表关联的传统查询方式

对于某些场景可能要利用到这个特性。

比如有模型集合 A，里面有关联 `a`, `a.b`, `a.c`，我们需要加载这些关联很简单，按以下的写法写就行。

```
$a = app(A::class)->hydrate(xx);

$a->load([
    'a',
    'a.b',
    'a.c'
]);
```

现在我们有另外一个模型集合 B，也存在这些关联 `a`, `a.b`, `a.c`，一般情况下，我们会和集合 A 的 load 操作分开。

但是这个时候就产生了重复的查询了。比如上面，就产生了 3 个重复的查询。


## 去除重复查询

一种方法是，把 A 集合和 B 集合中的 `a_id` 拿出来合并去数据库查询。但是这样操作起来相对麻烦，查询出来还要手动组装回去。

另外一种方法是，合并两个集合 A 和 B 得到一个新的 Eloquent 集合。然后使用 Eloquent Collection 的 load 方法一次性加载关联数据。这样就可以避免很多重复查询。

```
$c = $a->merge($b);

$c->load([
    'a',
    'a.b',
    'a.c'
]);
```

为什么行得通？

因为我们操作的是模型，一个个对象，在 load 操作执行的时候会设置对应对象的属性，而新集合持有的对象和原集合的对象是一样的，所以最终的结果就是：集合 A 和集合 B 里面的对象都获取到了关联数据。


## 这和 merge 有什么关系？

现在有另外一种场景，我们有一个集合 A 和一个 `a_id` 数组，但是我们也想一次性查询出集合 A 的关联和 `a_id` 数组的关联数据，避免重复查询。

我的做法是，根据 `a_id` 数组建立另一个模型 B 的集合，其外键 `a_id` 的值分别为 `a_id` 数组里的值。

```
$b = collect();

foreach($a_ids as $a_id) {
    $model = app(B::class);
    $model->a_id = $a_id;
    $b->push($model);
}

$c = $a->merge($b);

$c->load([
    'a',
    'a.b',
    'a.c'
]);
```

我们想要的结果是，也是没有重复查询。但是事与愿违，我们发现产生了一些重复的查询，这是为什么呢？

通过打印 `$c`，发现里面只有一个 B 模型的对象，其他的都没有了。这就是本文要探讨的问题，因为这些 B 模型对象没有设置主键，其主键都是 null，最终只有一个成功合并进去，即使这些模型的 `a_id` 属性不一样。



## 破解之道

现在我们知道了原因，也就不难解决了，因为这个模型 B 其实不是我们需要的数据，我们需要的只是其查询出来的关联数据，我们就给它们不一样的主键值就行了。

```
$key = 0;

foreach($a_ids as $a_id) {
    $model = app(B::class);
    $model->a_id = $a_id;
    $mode->{$model->getKeyName()} = $key++; // 设置不一样的主键值
    $b->push($model);
}
```

最后发现，没有了重复的查询。


## 如何拿出这些关联？

```
$a = $b->pluck('a');
$ab = $b->pluck('a.b');
$ac = $b->pluck('a.c');
```
