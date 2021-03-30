---
title: Laravel Indirect modification of overloaded property 报错原因
date: 2021-03-30 10:30:00
tags: [Laravel, PHP]
---

> 这种情况只在通过 `__get` 获取对象属性的时候才会出现。之前也写过一篇类似的文章，但是那里阐述的报错原因没那么直观，本文指出了 laravel 里面出现这个报错的常见原因。

在 Laravel 中，有时候我们需要修改关联的某一个属性的值。这个时候我们一般会使用 `$a->b->c = 1;` 这种形式。

这种形式一般情况下是没有问题的，但是如果在关联的值是 null 的时候，也就是没有对应的关联的时候就会报错。

而这个报错往往让人摸不着头脑，事实上原因很简单，只是设置了 null 的属性。

### 报错信息

```
Indirect modification of overloaded property A::$b has no effect
```

### 测试源码（lumen）

```PHP
<?php

use Illuminate\Database\Eloquent\Model;

require_once __DIR__ . '/bootstrap/app.php';

app()->boot();

/**
 * Class A
 * @property-read B b
 */
class A extends Model
{
    protected $table = 'as';

    public function b()
    {
        return $this->hasOne(B::class);
    }
}

class B extends Model
{
    protected $table = 'bs';
}

/** @var A $a */
$a = A::query()->first();

dump($a->b); // null
$a->b->id = 2; // Indirect modification of overloaded property A::$b has no effect

```

### 解决方法

1、如果我们允许这个值为 null，并且需要在其不为 null 的时候设置其值，则可以用 `optional` 方法：

```PHP
optional($a->b)->id = 2;
```

2、如果业务上是不会出现 null 的情况的，直接让它异常就好了，早点发现。
