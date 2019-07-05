---
title: Laravel Model - Indirect modification of overloaded property
date: 2018-07-05 18:00:00
tags: [Laravel, PHP]
---

在 Laravel 中，有时候我们想修改模型对象的某个属性的时候，可能会报错 `Indirect modification of overloaded property User::$info has no effect`。

这是因为我们给模型设置的属性获取我们通过属性的方式获取对象信息的时候，实际上是通过 Model 的 `__get` 魔术方法获取的。在模型的 $attributes 属性数组里面，保存了我们设置的对象属性，比如下面的操作：

定义模型

```php
class User extends \Illuminate\Database\Eloquent\Model
{}
```

```php
$user = new User;
$user->age = 26;
```

实际上是通过 Model 的 `__set` 方法在 Model 的 `$attributes` 数组属性里面加了一个属性：

```php
$user = new User;
$user->age = 26;

dd($user->toArray(), $user->getAttributes()); // ['age' => 26], ['age' => 26]
```

\Illuminate\Database\Eloquent\Model::__get 方法定义

```
/**
 * Dynamically retrieve attributes on the model.
 *
 * @param  string  $key
 * @return mixed
 */
public function __get($key)
{
    return $this->getAttribute($key);
}
```

前面我们设置了 `$user->age = 26;` 但是 `property_exists($user, 'age')` 会是 false。

我们获取这个属性的时候 `$user->age`，实际上等同于 `$user->__get('age')`，也就是说，我们想修改里面属性的时候比如 `$user->info['job'] = 'PHP';`，实际上等同于：

```php
$user->__get('info')['job'] = 'PHP';
```

也就是说，等同于：

```
$temp = $user->__get('info');
$temp['job'] = 'PHP';
```

也就是说，我们在设置有层级的属性的时候，中间产生了临时变量。

假设大家知道了值传递、引用传递的区别。
 
这个时候问题就来了，如果这个 `$user->info` 是一个数组，我们最终修改的是一个临时数组，而 `$user->info` 还是不变。这个时候就报错了，因为这个修改是完全没有意义的。

而如果 `$user->info` 是一个对象的时候，我们临时变量也是一个对象，而对象是通过引用传递的，我们修改了这个对象的时候，实际上也修改了 `$user->info`，因为这个临时变量指向了原始对象。


##### 完整例子

```php

$user = app(User::class);

$user->infoObj = new stdClass;
$user->infoObj->age = 26;
$user->infoObj->job = 'PHP';

// dd($user->toArray());

// Indirect modification of overloaded property User::$info has no effect
$user->info = [
    'age' => 26,
];
$user->info['job'] = 'PHP';
```
