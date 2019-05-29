---
title: Laravel 获取所有表名
date: 2018-07-26 15:16:00
tags: [Laravel, PHP]
---

```php
$tables = DB::connection()->getDoctrineSchemaManager()->listTableNames();
```

需要 `doctrine/dbal` 扩展，`Laravel` 本身也依赖这个扩展，所以无需额外安装
