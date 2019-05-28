---
title: Laravel5.1 eloquent with 通过闭包筛选特定 field 得不到结果的问题.md
date: 2018-01-06 09:35:00
tags: [Laravel, PHP]
---

`App\User`

```php
class User extends Model
{
    public function profile()
    {
        return $this->hasOne('App\UserProfile');
    }
}
```

使用 `with` 查询某个 `user` 及其的 `profile`:

```php
App\User::with([
    'profile' => function($query) {
        $query->select(['id']);
    }
])
->find(4)
->toArray();

```

上面的用法中，我们会发现，即使数据库有记录，`sql` 也记录了对应的查询语句，但是 `profile` 关联却是空的:

但是加上外键就可以得到正确结果了：

```php
App\User::with([
    'profile' => function($query) {
        $query->select(['id', 'user_id']); // 多了 user_id
    }
])
->find(4)
->toArray();

```

可以查找到正确的 `profile` 了。

这和 `Laravel` 框架的工作方式相关，我们先看看下面的例子：

我们使用 `DB::listen` 方法去记录相关的 `sql` 语句


我们查看 log 可以发现有以下语句：
```sql
select * from `users` where `id` in (?, ?) [3,4]
select * from `profiles` where `profiles`.`user_id` in (?, ?) [3,4]
```

我们可以明显发现，`laravel` 对于 `user` 和 `profile` 是独立查询的，

也就是说会得到两个集合，一个是 `User`、一个是 `Profile`，但是这并不是我们想要的结果，我们需要的结果是，只有一个 `User` 集合， 并且这个 `User` 集合里面有 `Profile` 关联。



但是结果就是这样，如果是你，你会怎么把这些数据关联起来呢？

对了，我们定义关联的时候不是定义了它们的关联方式么？

上面的 `hasMany` 方法默认第二第三个参数其实就是这两个集合建立关联的关键，第三个参数 `user_id`、第四个参数 `id`；这样一来我们就可以通过比较 `Profile` 的 `user_id` 和 `User` 里面的 `id`，如果相等，则这个 `Profile` 是属于这个 `User` 的，我们就把该 `Profile` 放进 `User` 的 `profile` 关联中，最后就得到我们想要的结果了。


用 `xdebug` 证实一下我们的想法：

![match](/images/1-min.png)
![getRelationValue](/images/2.png)
![matchOneOrMany](/images/3.png)

如我们所想的那样，图一的 `match` 方法，顾名思义就是匹配了，通过 `user` 模型集合和 `profile` 模型集合进行匹配。

图二，也证实了我们模型建立关联需要通过关联中外键的值得想法。

图三，是通过获取 `user` 的 `localkey`，也就是 `id` 的值，来查找 `$dictonary` 中是否有对应的值，`buildDictonary` 方法会建立一个关联数组，`key` 是 `user_id`（外键）的值，值是关联的数据。这样一样，由于我们没有把 `user_id` 也 `select` 出来，最后得到的 `$dictonary` 的结构并不是预期的那样：

![xdebug](/images/4.png)

其实我们本来是想要得到下面的这种：

```php
[
  3 => xxx(UserProfile对象)       // 3 是关联的 user_id
]
```

但是我们得到的却是，所有的 `UserProfile` 都在一个嵌套的数组里面了，这样一来，下面的 `getRelationValue` 得到的结果自然就是空的了。


#### 总结

好了，总结一下，就是：`Laravel` 先查询主要的数据(不带 `with`)，查询完了之后，取出其中的 `id` 列数组(不一定都是id啊，只是举个例子)，将这个数组作为条件去查找关联，有多少个关联就会再去查找多少次，查找完关联之后通过得到的结果的主键和关联数据的外键比对，相等则建立关联。

总结：在关联筛选 `field` 的时候，也必须要把关联的外键写进去，否则，即使产生了正确的 `sql` 语句，但是它们建立不了关联，通过 `$user->profile` 得到的还是一个空集合。
