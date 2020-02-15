---
title: jenssegers mongodb Model 如何正确地手动设置模型关联
date: 2020-02-15 18:03:00
tags: [Laravel, PHP]
---

> 有时候我们操作一个模型的时候，其关联模型已从其他地方查询出来，现在想要给这个模型赋予这个关联，以防止在访问这个关联的时候再去查询数据库。


## 例子

我们现在有一个查询出来的关联 `sku`, 现在需要把它赋给 `$cart` 模型对象，这样我们使用 `$cart->sku` 来访问 `sku` 属性， 的时候就不会再去查询数据库了。

我们试试这样 

```
<?php

require_once __DIR__ . '/bootstrap/app.php';

enable_mysql_log();

$cart = app(\Modules\Order\Models\Cart::class); // jenssegers mongodb Model
$sku = app(\Modules\Product\Models\Sku::class)->first();

$cart->sku = $sku;

dump($cart->sku); // null, 这里拿不到 sku，当然这里是因为 cart 没有设置关联键的值，如果有设置的话，会进行数据库查询
```

这说明实际上，我们在使用 `$cart->sku` 的时候实际上并不能获取到 `$cart->sku = $sku;` 这里设置的 `sku`。


## 源码剖析

\Jenssegers\Mongodb\Eloquent\Model::getAttribute

```
/**
 * @inheritdoc
 */
public function getAttribute($key)
{
    if (!$key) {
        return;
    }

    // Dot notation support.
    if (Str::contains($key, '.') && Arr::has($this->attributes, $key)) {
        return $this->getAttributeValue($key);
    }

    // This checks for embedded relation support.
    if (method_exists($this, $key) && !method_exists(self::class, $key)) {
        return $this->getRelationValue($key);
    }

    return parent::getAttribute($key);
}
```

我们获取模型属性的时候会先调用这个方法，我们发现，`jenssegers mongodb Model` 覆盖了 Laravel Model 的 getAttribute 方法，因为在 `Cart` 模型里面定义了 `sku` 关联，因此这里实际上返回的是：

```
return $this->getRelationValue($key);
```

接下来再看 `getRelationValue`，

```
/**
 * Get a relationship.
 *
 * @param  string  $key
 * @return mixed
 */
public function getRelationValue($key)
{
    // If the key already exists in the relationships array, it just means the
    // relationship has already been loaded, so we'll just return it out of
    // here because there is no need to query within the relations twice.
    if ($this->relationLoaded($key)) {  // 这个条件会不成立
        return $this->relations[$key];
    }

    // If the "attribute" exists as a method on the model, we will just assume
    // it is a relationship and will load and return results from the query
    // and hydrate the relationship's value on the "relationships" array.
    if (method_exists($this, $key)) {
        return $this->getRelationshipFromMethod($key);
    }
}
```

在这个方法中 `$this->relationLoaded($key)` 这个条件不成立，因此导致了实际运行了 `$this->getRelationshipFromMethod($key);`，这就导致了再次查询数据库。


## 解决方法

翻看源码，发现这个条件实际上是 `array_key_exists($key, $this->relations);`，但是很遗憾，我们使用手动设置关联的方式的时候，`Cart` 模型实际上把 `sku` 当作一个普通的属性，而不是 `relation`，因此在 `$cart` 的 `relations` 属性中并没有 `sku`。

唯一的方法就是通过调用 `setRelation` 来显式设置模型关联：

```
/**
 * Set the specific relationship in the model.
 *
 * @param  string  $relation
 * @param  mixed  $value
 * @return $this
 */
public function setRelation($relation, $value)
{
    $this->relations[$relation] = $value;

    return $this;
}
```

## Laravel 的 Model 有没有这个问题

没有。

Laravel 的 Model 会先去模型的 `attributes` 里查看是否存在对应的属性，最后才看关联有没有这个关联。
