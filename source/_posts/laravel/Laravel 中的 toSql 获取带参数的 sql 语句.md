---
title: Laravel 中的 toSql 获取带参数的 sql 语句
date: 2018-06-27 10:40:00
tags: [Laravel, PHP]
---

默认情况下，`toSql` 获取到的 `sql` 里面的参数使用 "?" 代替的，如下：

```php
DB::table('users')->where('id', 1)->toSql();
```

 获取到的 `sql` 语句是：
 
 ```php
 select * from `users` where `id` = ?
 ```
 
有时候我们想要得到具体的语句，可以利用 `builder` 的 `getBindings` 方法：
 
```php
$builder = DB::table('users')->where('id', 1);
$bindings = $builder->getBindings();
$sql = str_replace('?', '%s', $builder->toSql());
$sql = sprintf($sql, ...$bindings);
dd($sql);
```

获取到的 `sql` 语句是：

```php
select * from `tb_user` where `id` = 1
```

如果经常使用可以考虑使用 `Builder` 的 `macro` 方法加进 `Builder` 里面：

```php
\Illuminate\Database\Query\Builder::macro('sql', function () {
    $bindings = $this->getBindings();
    $sql = str_replace('?', '%s', $this->toSql());
 
    return sprintf($sql, ...$bindings);
});
dd(DB::table('users')->where('id', 1)->sql());
```
