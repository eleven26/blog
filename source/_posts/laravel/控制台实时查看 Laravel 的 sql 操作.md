---
title: 控制台实时查看 Laravel 的 sql 操作
date: 2018-09-05 15:53:00
tags: [Laravel, PHP]
---

[listen-sql](https://github.com/eleven26/listen-sql) 一个在控制台看到实时 sql 操作的工具

在 Laravel 中打印 sql，以往的做法往往是，通过 `DB::listen` 监听，然后通过 `Log::info` 写入到 log 中。

这样写入的 log，我们想查看往往是去 storage 文件夹下找到当天的 log 文件，然后打开。有个不好的地方是，如果在编辑器打开，往往不会实时更新。请求完之后，可能需要切到其他 tab 再切换回来才会更新。同时，太多的 sql 日志会和其他 log 混杂在一起，会显得有些混乱。

除此之外，也可以 `tail -f storage/logs/xx.log` 来实时查看 log 的输出。这样有个不好的地方是，如果在 `config/app.php` 定义了 `log => 'daily'`，每天都要输入一个新的文件名。

现在，我们可以只使用一个命令来实现监听应用里的 sql 操作。

![控制台实时查看 sql](https://cdn.learnku.com/uploads/images/201909/05/7747/syTIBmM7od.png!/fw/1240)


## 安装

1.通过 [composer](https://getcomposer.org/) 安装 ([eleven26/listen-sql](https://packagist.org/packages/eleven26/listen-sql))。

```bash
composer require "eleven26/listen-sql:~1.0.2"
```

2.注册 Service Provider

- `Laravel`: 修改文件 `config/app.php`，`Laravel 5.5+` 不需要
    ```php
    'providers' => [
        //...
        Eleven26\ListenSql\ListenSqlServiceProvider::class,
    ],
    ```

- `Lumen`: 修改文件 `bootstrap/app.php`
    ```php
    $app->register(Eleven26\ListenSql\ListenSqlServiceProvider::class);
    ```


## 使用

```php
php artisan listen-sql:start
```

到这一步，去页面刷新的时候，就可以在控制台看到 `sql` 语句了
