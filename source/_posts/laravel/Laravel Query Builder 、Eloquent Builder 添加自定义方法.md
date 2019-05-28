---
title: Laravel Query Builder 、Eloquent Builder 添加自定义方法
date: 2018-04-17 22:36:00
tags: [Laravel, PHP]
---


#### 使用 `Macroable` 给 `Builder` 添加自定义方法

`Laravel` 中提供了 `Macroable` 的 `trait`，之前一直没有想过可以用上这个东西。

最近才想到可以这么做，源码看这里：[https://github.com/laravel/framework/blob/5.6/src/Illuminate/Support/Traits/Macroable.php](https://github.com/laravel/framework/blob/5.6/src/Illuminate/Support/Traits/Macroable.php)

目前用的是 `5.1` 版本，应该 `5.1` 版本以后的都有这个特性。


当然，你也可以在 `Model` 基类里面添加自定义方法，但是这样添加的方法 `DB` 类是用不了的。

但是使用 `macro` 定义到 `Builder` 的方法可以同时在 `DB` 类和 `Eloquent Model` 中使用（具体不赘述，自己看源码，就是利用 `__call`、`__callStatic` 这些魔术方法）。


##### 定义

使用方法：调用 `Macroable` 的 `macro` 方法，绑定一个自定义方法到 `Builder` 类中，如：

```php
\Illuminate\Database\Query\Builder\Builder::macro('active', function () {
    return $this->where('status', 1);
});

```

##### 使用

调用方法是（使用 `DB` 类）：

```php
DB::table(xxx)->active()->get();
```

或者（使用 `Eloquent Model`）：

```php
\App\Model\User::active()->first();
```

#### 放在哪里？

至于我们应该把上面那几行放哪里？

个人觉得一个比较好的地方是 `Providers` 中，在 `app/Providers` 下面新建一个 `Provider`，把 `macro` 调用放到 `Provider` 的 `register` 方法中。如：

```php
<?php
 
namespace App\Providers;
 
use Illuminate\Database\Query\Builder;
use Illuminate\Support\ServiceProvider;
 
/** @mixin \Illuminate\Database\Eloquent\Builder */
class DatabaseServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        Builder::macro('active', function () {
            return $this->where('status', 1);
        });
    }
}
```

当然，加了 `Providers` 之后还要在 `config/app.php` 中配置这个 `Provider`。

就这样。


还有个问题是，这样添加的方法 `ide` 无法识别，我们这时候就可以使用 `@method` 了，如：

```php
@method $this active()
```

可以使用命令把这些 `@method` 写入到 `Builder` 头部。

