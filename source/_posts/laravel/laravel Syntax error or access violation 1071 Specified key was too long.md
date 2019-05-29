---
title: laravel Syntax error or access violation: 1071 Specified key was too long; max key length is 767 bytes (SQL: alter table users add unique users_email_unique(email)
date: 2018-11-22 19:32:00
tags: [Laravel, PHP]
---

AppServiceProvider

```php
use Illuminate\Support\Facades\Schema;
 
public function boot()
{
    Schema::defaultStringLength(191);
}
```
