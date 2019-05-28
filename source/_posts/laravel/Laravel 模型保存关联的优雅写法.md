---
title: Laravel 模型保存关联的优雅写法
date: 2017-12-24 10:24:00
tags: [Laravel, PHP]
---

模型定义

`App\User`

```php
class User
{
    public function customer()
    {
        return $this->hasOne('App\Customer');
    }
}
```

`App\Customer`

```php
class Customer
{
    public function user()
    {
        return $this->belongsTo('App\User');
    }
}
```

需要注意的是 `associate` 方法是 `BelongsTo` 类才有，所以正确的调用方法如下：

```php
use App\User;
use App\Customer;

$user = new User($data);
$customer = new Customer($customerData);
 
$customer->user()->associate($user);
$customer->save();
```

与此相反的方法是 `disassociate` 方法：取消两个模型之间的 `belongsTo` 关联

```php
$customer->user()->disassociate();
$customer->save();
```

此方法需要注意的是，`disassociate` 并不会删除记录，只是更新关联的字段为 `null`。上面这种操作会把 `customer` 表的 `user_id` 设置为 `null`。


附(通过关联的模型保存)：
```php
use App\User;
use App\Customer;

$user = new User($data);
$user->save();
 
$customer = new Customer($customerData);
$user->customer()->save($customer);
```
