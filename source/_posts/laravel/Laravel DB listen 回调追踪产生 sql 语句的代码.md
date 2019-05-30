---
title: Laravel DB listen 回调追踪产生 sql 语句的代码
date: 2018-12-24 16:54:00
tags: [Laravel, PHP]
---

```php
\DB::listen(function (QueryExecuted $sql) {
     \Log::info($sql->sql);
     \Log::info((new \Exception())->getTraceAsString());
});
```
