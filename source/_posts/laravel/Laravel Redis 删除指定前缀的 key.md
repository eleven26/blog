---
title: Laravel redis 删除指定前缀的 key
date: 2019-04-22 16:43:00
tags: [Laravel, PHP]
---

```php
// 前缀
$prefix = 'abc';
 
// 需要在前面连接上应用的缓存前缀
$keys = app('redis')->keys(config('cache.prefix') . $prefix . '*');
 
app('redis')->del($keys);
```
