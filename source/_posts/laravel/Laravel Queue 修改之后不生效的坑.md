---
title: Laravel Queue 修改之后不生效的坑
date: 2018-06-14 12:37
tags: [Laravel, PHP]
---

其实官方文档有说，只是没看仔细。

 
正常情况下，修改 `php` 代码是不用重启什么东西的，

但是 `Laravel` 中的 `job` 不一样，

如果不用 `php artisan queue:restart`，新 `dispatch` 的 `job` 跑的还会是原来的代码。

也就是说，如果发现 `job` 有 `bug`，改了之后可能不会生效，除非有重启过队列。

如果使用 `supervisor`，可以使用 `supervisor restart xxx` 来进行重启。

 

个人觉得原因是，运行 `php artisan queue:work` 或 `php artisan queue:listen` 之后，

内存中的代码是运行时候的代码，是常驻内存的，

`job` 的保存是序列化保存的，在 队列运行到对应的 `job` 的时候，反序列化出来的还是旧驻留内存的 `job`。


所以，队列有些奇怪的 `bug` 发生的时候，或者修改过代码之后，需要重启一下队列。


