---
title: Lumen passport Call to undefined method Application group
date: 2018-11-18 20:21:00
tags: [Laravel, PHP]
---

* 解决方法：[https://github.com/dusterio/lumen-passport/issues/69](https://github.com/dusterio/lumen-passport/issues/69)

* `public function boot() { LumenPassport::routes($this->app->router); }`

* 该方法的参数应该传递 `$this->app->router` 而不是 `$this->app`
