---
title: Laravel 添加自定义 Provider 配置之后不生效的问题
date: 2018-04-21 10:53:00
tags: [Laravel, PHP]
---

有可能是配置缓存导致的，

运行：

```bash
php artisan config:clear
```

可清除配置缓存，配置缓存保存在 `bootstrap/cache/config.php`，可以直接去那文件夹看看是不是缓存导致的。

还有另外一个缓存文件 `bootstrap/cache/services.json`，上面的命令只是清除配置，下面的命令可以同时清除这两个缓存文件。

```bash
php artisan clear-compiled
```


```php
php artisan config:clear
php artisan clear-compiled // 还不行再运行下面的
php artisan optimize
```
