---
title: Lumen 增加视图路径
date: 2019-01-19 13:18:00
tags: [Laravel, PHP]
---

在 `ServiceProvider` 里面加上

```php
app('view')->addLocation(module_path('Develop') . '/resources/views');
```
