---
title: Dingo api 处理请求机制
date: 2019-08-20 13:53:00
tags: [dingo]
---

Lumen 使用 dingo/api 的第一步便是 `$app->register(Dingo\Api\Provider\LumenServiceProvider::class);`，所以就从这个文件说起吧。

这个文件的父类是 `\Dingo\Api\Provider\DingoServiceProvider`，我们可以在这个文件中发现其注册了很多的东西到 App 容器里面，其中也有很多单例的对象等等等等，

但是，这些都不重要，重要的是 **\Dingo\Api\Provider\LumenServiceProvider::boot**  方法里面的处理:

```php
/**
 * Boot the service provider.
 *
 * @return void
 */
public function boot()
{
    parent::boot();

    // 1. 加载 dingo/api 配置
    $this->app->configure('api');

    // 2. 获取 app 容器反射对象
    $reflection = new ReflectionClass($this->app);

    // 3. 拿出 app 容器中定义的中间件, 传递进 \Dingo\Api\Http\Middleware\Request 中间件中
    $this->app[Request::class]->mergeMiddlewares(
        $this->gatherAppMiddleware($reflection)
    );

    // 4. 把 \Dingo\Api\Http\Middleware\Request 中间件加到 app 中间件处理链起始位置，
    //    这样一来，请求会先经过 \Dingo\Api\Http\Middleware\Request 中间件处理，然后才是我们使用 `app()->middleware` 定义的中间件。
    $this->addRequestMiddlewareToBeginning($reflection);

    // Because Lumen sets the route resolver at a very weird point we're going to
    // have to use reflection whenever the request instance is rebound to
    // set the route resolver to get the current route.
    $this->app->rebinding(IlluminateRequest::class, function ($app, $request) {
        $request->setRouteResolver(function () use ($app) {
            $reflection = new ReflectionClass($app);

            $property = $reflection->getProperty('currentRoute');
            $property->setAccessible(true);

            return $property->getValue($app);
        });
    });

    $this->app->routeMiddleware([
        'api.auth' => Auth::class,
        'api.throttle' => RateLimit::class,
        'api.controllers' => PrepareController::class,
    ]);
}
```

到此为止，我们已经知道，我们的请求会先经过 `\Dingo\Api\Http\Middleware\Request` 中间件处理，这是很关键的一步，接下来，我们看看这个中间件做了什么:

```php
/**
 * Handle an incoming request.
 *
 * @param \Illuminate\Http\Request $request
 * @param \Closure                 $next
 *
 * @return mixed
 */
public function handle($request, Closure $next)
{
    try {
        // 验证是否是 dingo 请求(前缀匹配上了 API_PREFIX 或者域名匹配上 API_DOMAIN)，若不是会直接传递给下一个中间件
        // (详情: \Dingo\Api\Provider\HttpServiceProvider::registerHttpValidation)
        // 若是:
        //    1. dingo 接管异常处理
        //    2. 转换 \Illuminate\Http\Request 为 \Dingo\Api\Http\Request
        //    3. 重新绑定 app 容器的 request 对象为 Dingo Request， 然后把 $request 传递给其他 app 容器中定义的中间件处理
        //    4. 异常的时候返回异常响应对象
        if ($this->validator->validateRequest($request)) {
            $this->app->singleton(LaravelExceptionHandler::class, function ($app) {
                return $app[ExceptionHandler::class];
            });

            $request = $this->app->make(RequestContract::class)->createFromIlluminate($request);

            $this->events->fire(new RequestWasMatched($request, $this->app));

            // 这个方法效果是所有中间件处理完之后才会执行控制器里面对应的方法
            return $this->sendRequestThroughRouter($request);
        }
    } catch (Exception $exception) {
        $this->exception->report($exception);

        return $this->exception->handle($exception);
    }

    return $next($request);
}
```

最后一步，也是很关键的一步:

在所有中间件处理完之后，会把 $request 传递给 `\Dingo\Api\Routing\Router::dispatch` 处理，

处理完成后，使用 `\Dingo\Api\Routing\Router::prepareResponse` 处理控制器方法返回的响应结果

这是 dingo/api transformer include 操作实际执行的地方，

也许我们都曾经有过这样的疑惑：include 到底是在哪里进行数据库查询的？ 其实是在这个方法里面执行的，具体细节有兴趣的执行研究。

这也算是 dingo 的一个特性了，但是 include 有太多坏处了，例如 N+1，但是也还是有解决方法的，不用 include... 还有嵌套 include 导致代码难以维护等等
