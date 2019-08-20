---
title: Laravel-s Dingo (Lumen) 支持
date: 2019-08-20 11:40:00
tags: [Laravel, Dingo, laravel-s, Lumen]
---

> 原因: 项目重度依赖 Dingo, 直接放弃 Dingo 的代价太大

> 有兴趣了解原理的可以看 [Dingo api 处理请求机制](https://blog.baiguiren.com/2019/08/20/laravel/Dingo%20api%20%E5%A4%84%E7%90%86%E8%AF%B7%E6%B1%82%E6%9C%BA%E5%88%B6/)

## 各个依赖版本

* [laravel-s(3.5.8)](https://github.com/hhxsv5/laravel-s/releases/tag/v3.5.8)

* swoole 4.3.3

* php-7.1.14

* Lumen 5.5.38

* Dingo/api 2.0.0-alpha2.2


## 步骤

#### 修改 Laravel.php 的 handleDynamic 方法，如下

这一步需要另外 copy 一份 `Laravel.php` 出来，假设放在 `App\Swoole\Laravel.php`，

App\Swoole\Laravel.php

```php
... // 其他内容和原文件一致

public function handleDynamic(IlluminateRequest $request)
{
    ob_start();

    if ($this->isLumen()) {
        // dingo router 路由处理
        $response = app('Dingo\Api\Http\Middleware\Request')->handle($request, function ($request) {
            return $request;
        });
        
        // 不是 dingo router 里面定义的路由
        if ($response instanceof \Illuminate\Http\Request) {
            $response = $this->app->dispatch($response);
        }
        
        if ($response instanceof SymfonyResponse) {
            $content = $response->getContent();
        } else {
            $content = (string)$response;
        }

        if ($response instanceof \Symfony\Component\HttpFoundation\Response) {
            $this->reflectionApp->callTerminableMiddleware($response);
        }
    } else {
        $response = $this->kernel->handle($request);
        $content = $response->getContent();
        $this->kernel->terminate($request, $response);
    }

    // prefer content in response, secondly ob
    if (!($response instanceof StreamedResponse) && strlen($content) === 0 && ob_get_length() > 0) {
        $response->setContent(ob_get_contents());
    }

    ob_end_clean();

    return $response;
}

... // 其他内容和原文件一致
```

原来的处理方式: `$response = $this->app->dispatch($request);` 这个处理方式会只使用 Lumen 的 `app()` 来处理 HTTP 请求，第一次请求正常，但是后续请求没什么意外是无法正常处理的。

新的处理方式: 使用 `app('Dingo\Api\Http\Middleware\Request')` 来处理 HTTP 请求，这是正常情况下 Dingo 处理请求的第一个经过的中间件，如果请求之后返回的还是一个 `Request`，而不是 `Response`，说明这不是使用 `app('api.router')` 定义的路由，退化为使用 `$response = $this->app->dispatch($request);` 处理该请求。


#### 自动加载处理

然后在 `bin/laravels.php` $basePath 后面加上:

```php
require_once $basePath . '/app/Swoole/Laravel.php';
```

目的: 自动加载机制在加载 `Laravel` 类的时候加载的是自定义的 `Laravel.php`，而不是原来的 [Laravel.php](https://github.com/hhxsv5/laravel-s/blob/v3.5.8/src/Illuminate/Laravel.php) 这样我们的修改就起效了。


#### Dingo 控制器在请求结束清理

添加文件 app/Cleaners/ControllerCleaner.php

```php
<?php

namespace App\Cleaners;

use Hhxsv5\LaravelS\Illuminate\Cleaners\CleanerInterface;
use Illuminate\Container\Container;

class ControllerCleaner implements CleanerInterface
{
    public function clean(Container $app, Container $snapshot)
    {
        $ref = new \ReflectionObject($app);
        $property = $ref->getProperty('instances');
        $property->setAccessible(true);
        $instances = $property->getValue($app);

        foreach ($instances as $key => $instance) {
            if ($instance instanceof \Laravel\Lumen\Routing\Controller) {
                $instance = null;
                app()->offsetUnset($key);
            }
        }
    }
}

```

然后在 `config/laravels.php` 中的 `cleaners` 中加上该 Cleaner:

```php
'cleaners' => [
    App\Cleaners\ControllerCleaner::class, // 单例控制器清除
],
```


#### Dingo 中间件处理

新增文件 App\Cleaners\MiddlewareCleaner.php

```php
<?php

namespace App\Cleaners;

use Dingo\Api\Http\Middleware\Request;
use Hhxsv5\LaravelS\Illuminate\Cleaners\CleanerInterface;
use Illuminate\Container\Container;

class MiddlewareCleaner implements CleanerInterface
{
    public function clean(Container $app, Container $snapshot)
    {
        $reflect = new \ReflectionObject($snapshot);
        $property = $reflect->getProperty('middleware');
        $property->setAccessible(true);

        $middleware = $property->getValue($snapshot);

        // 移除 Dingo\Api\Http\Middleware\Request，防止死循环
        $middleware = array_values(array_diff($middleware, [Request::class]));

        app(Request::class)->setMiddlewares($middleware);
    }
}

```

然后在 `config/laravels.php` 的 cleaners 中加上该 cleaner:

```php
'cleaners' => [
    App\Cleaners\MiddlewareCleaner::class, // 单例控制器清除
],
```

目的: 这个命名不是很准确地说明它的作用，其实这个文件的作用是把被 dingo 清除的中间件还原回来，这样后续的请求才会有第一次请求的中间件。

（dingo 会在其处理的周期内通过反射把 app 容器内的中间件清除，会导致后续请求没有经过这些中间件。）

这里说的中间件是定义在 `app()` App 容器中的中间件:

bootstrap/app.php

```php
$app->middleware([
    XxxMiddleware::class,
]);
```

php-fpm 环境下，由于每个请求相互之间不会相互影响，因此不会有问题，而在 laravel-s 环境下，请求结束之后，请求 context 不会被完全销毁，有可能会对后续请求造成影响。

