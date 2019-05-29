---
title: Laravel 嵌套事务
date: 2018-11-10 09:29:00
tags: [Laravel, PHP]
---

* 什么是嵌套事务? 一般情况下我们都是一个 `begin`, 一个 `commit` 或 `rollBack`, 但是有可能我们有种场景需要 `begin` 然后在事务里面再开一个事务, 这就是嵌套事务.

* `MySQL` 嵌套事务支持
    - `MySQL` 里面有个 `savepoint` 关键字, 可以模拟嵌套事务, 但事实上并不是真正的嵌套事务, 仍然是一个事务.
    - `savepoint` 的用处, 我们可以回滚事务内的部分修改

* `Laravel` 嵌套事务使用
    - `DB::beginTransaction()` 里面再次使用 `DB::beginTransaction()` 即可, 但需要注意开启事务的次数需要和 `commit` 或 `rollBack` 的次数对应

