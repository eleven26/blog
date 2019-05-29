---
title: Lumen 单元测试的一些问题
date: 2018-09-14 22:47:45
tags: [Laravel, PHP]
---

#### 一个 test 多个请求

如 `$this->post`，然后又 `$this->post`，我们会发现第二个请求中的请求参数是和第一个请求的参数是完全一样的，
然后在 `Controller` 里面通过 `spl_object_hash` 方法发现两个请求的 `request` 实例是一样的，
应该是第二个请求发起的时候，`request` 不再实例化，直接使用了上一次请求的 `request` 实例。

这种情况我们可以用过 `request` 实例的 `replace` 方法，替代掉 `request` 实例的请求参数，这样我们第二个请求就可以按照我们预期地跑了。


```php
$this->post('xxx', ['a' => 1]);
 
app('request')->replace([
    'b' => 2   
]);
 
$this->post('yyy');
```

#### 使用 `mock` 的时候，`mock` 一个不存在的方法不会报错

其实这也算是 `mock` 本身要实现的功能，但是如果我们可能在调用多个对象的方法的时候会混淆，`mock` 了一个错误的对象的方法，但实际上应该是 `mock` 另外一个。

如果我们 `mock` 了之后，对象方法表现还是原来的样子就应该考虑一下是不是 `mock` 了一个错误的对象。


#### 不能对 `private` `final` `static` 方法进行 `mock`，需要对 `mock` 的对象调用 `setMethods`，说明我们要对哪些方法进行 `mock`，否则可能会报错。
