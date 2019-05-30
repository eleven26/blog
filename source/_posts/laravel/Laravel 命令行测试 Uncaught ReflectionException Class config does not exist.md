---
title: Laravel DB listen 回调追踪产生 sql 语句的代码
date: 2019-03-05 10:37:00
tags: [Laravel, PHP]
---

```php
require __DIR__ . '/vendor/autoload.php';
$app = require_once __DIR__ . '/bootstrap/app.php';

config('any');
```

![x](/images/14.png)

解决办法：

```php
require __DIR__ . '/vendor/autoload.php';
$app = require_once __DIR__ . '/bootstrap/app.php';

(new \Illuminate\Foundation\Bootstrap\LoadConfiguration)->bootstrap($app);
config('any');
```

使用 ` (new \Illuminate\Foundation\Bootstrap\LoadConfiguration)->bootstrap($app);` 加载配置
