---
title: laravels + dingo + lumen 不支持多版本解决方案
date: 2018-07-04 16:44:00
tags: [laravel-s, dingo, lumen]
---

自定义一个 Dispatcher

\App\Dispatcher.php

```php
<?php

namespace App;

use Dingo\Api\Http\Request;

class Dispatcher
{
    /**
     * Dispatch a request.
     *
     * @param $request
     * @return mixed
     */
    public function dispatch($request)
    {
        $version = Request::getAcceptParser()->parse($request)['version'];

        return app('api.router.adapter')->dispatch($request, $version);
    }
}
```

修改 `\Hhxsv5\LaravelS\Illuminate\Laravel::handleDynamic` 方法，把 `$response = $this->app->dispatch($request);` 替换成 `$response = app(\App\Dispatcher::class)->dispatch($request);`。

##### 原因

使用 `$response = $this->app->dispatch($request);` 的时候，实际上没有处理 http 头里面的版本信息

源码：
[https://github.com/hhxsv5/laravel-s/blob/v3.5.7/src/Illuminate/Laravel.php#L146](https://github.com/hhxsv5/laravel-s/blob/v3.5.7/src/Illuminate/Laravel.php#L146)
