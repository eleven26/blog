---
title: Laravel 模型事件 updated 触发条件
date: 2019-04-22 16:43:00
tags: [Laravel, PHP]
---

* 只有 `$sku->{attribute} != $sku->getOriginal({attribute})` 不一致的时候才会触发
　　`getDirty()` 不为空的时候才触发, 而且不会比较数据类型(判断是否 `dirty` 使用的是 `==` 而不是 `===`)

* 直接 `app(Model::class)->where()->update()` 不会触发
　　`$sku = app(Sku::class), $sku->has_stock = 1; $sku->save()` 这样才会触发

[https://github.com/laravel/framework/issues/11777#issuecomment-170384117](https://github.com/laravel/framework/issues/11777#issuecomment-170384117)
