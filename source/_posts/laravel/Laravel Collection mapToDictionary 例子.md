---
title: Laravel Collection mapToDictionary 例子
date: 2019-01-04 11:26:00
tags: [Laravel, PHP]
---

源码

![x](/images/12.png)

示例

```php
<?php
 
require __DIR__ . '/bootstrap/app.php';
 
$arr = [
    [
        'name' => 'John',
        'age' => 23
    ],
    [
        'name' => 'Neo',
        'age' => 25
    ],
    [
        'name' => 'John',
        'age' => 24
    ]
];
 
$v = collect($arr)->mapToDictionary(function ($v) {
    // 主要作用: 可以自定义每一项的 key value
    return [$v['name'] => $v];
})->toArray();
 
// $v 与 $v1 相等
$v1 = collect($arr)->groupBy('name')->toArray();
 
dd($v);
```

输出

![x](/images/13.png)
