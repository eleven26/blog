---
title: Lumen 使用 Dingo API 在 PHPUnit 中 404 的解决方法, 以及鉴权问题
date: 2018-09-13 14:24:00
tags: [Laravel, PHP]
---

#### `phpunit.xml` 中添加 `dingo` 相关配置

```php
<env name="API_STANDARDS_TREE" value="x"/>
<env name="API_SUBTYPE" value="prime"/>
<env name="API_VERSION" value="v1"/>
<env name="API_DEFAULT_FORMAT" value="json"/>
<env name="API_STRICT" value="false"/>
<env name="API_PREFIX" value="api"/>
```


#### `dingo` 的路由配置文件包含不能使用 `require_once`。

否则, 可能会出现一种情况是, `phpunit` 中第一个请求成功了, 但是后面的请求都 404。

这里涉及到的一个知识点是：laravel 或 lumen `phpunit` 中每一个 `test` 都会使用独立的 `Application` 以及 `TestCase` 实例来运行，我们知道，框架在初始化的时候会加载路由的配置，如果我们的进程只跑一遍，这样其实没什么问题。

但是，`phpunit` 中的 `TestCase` 是，一个进程，然后一个 `TestCase` 里面有多少 `test` 方法，就会进行多少次框架的初始化操作，这样问题就出现了，后续的请求中，`require_once` 不能再读取到任何配置信息，因此导致了后续请求全部 404 了。

所以需要使用 `require` 代替 `require_once`。

知识点：同一个进程中，`require_once` 只有第一次 `require` 会执行文件里面的内容，后续 `require` 不再加载文件。


#### 在测试中添加以下方法调用

* `phpunit.xml` 中添加 `dingo` 相关配置

```php
<env name="API_STANDARDS_TREE" value="x"/>
<env name="API_SUBTYPE" value="prime"/>
<env name="API_VERSION" value="v1"/>
<env name="API_DEFAULT_FORMAT" value="json"/>
<env name="API_STRICT" value="false"/>
<env name="API_PREFIX" value="api"/>
```

* `dingo` 的路由配置文件包含不能使用 `require_once`。否则, 可能会出现一种情况是, `phpunit` 中第一个请求成功了, 但是后面的请求都 404

* 在测试中添加以下方法调用

```php
public function setUp()
{
    parent::setUp();
 
    // 跳过 api 验证
    $user = \Modules\User\Models\User::with(['company'])->find(61339);
    $this->actingAsApi($user);
}
 
/**
 * Sets the API user.
 *
 * @param  mixed  $user
 * @return $this
 */
protected function actingAsApi($user)
{
    // mock service middleware
    /** @var Mockery\Mock $auth */
    $auth = Mockery::mock('Dingo\Api\Http\Middleware\Auth[handle]',
        [
            Mockery::mock('Dingo\Api\Routing\Router'),
            Mockery::mock('Dingo\Api\Auth\Auth'),
        ]);
    $auth->shouldReceive('handle')
        ->andReturnUsing(function ($request, \Closure $next) {
            return $next($request);
        });
    $this->app->instance('Dingo\Api\Http\Middleware\Auth', $auth);
    $auth = Mockery::mock('Dingo\Api\Auth\Auth[user]',
        [
            app('Dingo\Api\Routing\Router'),
            app('Illuminate\Container\Container'),
            [],
        ]);
    $auth->shouldReceive('user')
        ->andReturnUsing(function () use ($user) {
            return $user;
        });
    $this->app->instance('Dingo\Api\Auth\Auth', $auth);
 
    return $this;
}
```
