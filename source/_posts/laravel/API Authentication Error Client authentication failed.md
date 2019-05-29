---
title: API Authentication Error: {"error":"invalid_client","message":"Client authentication failed"}
date: 2018-11-18 21:16:00
tags: [Laravel, PHP]
---

解决方法：[https://github.com/laravel/passport/issues/221](https://github.com/laravel/passport/issues/221)

In your oauth_clients table, do the values you have above exist exactly as you have them in your database?

oauth_clients

```php
id - 3
secret - 3TfJGj4rrvOQvjZkI8dDqx78ouH99F2DuIMKHoKH
redirect - http://consumer.dev/callback
```

If it doesn't exist exactly like that in the database for passport.dev then it will throw the invalid_client error. Please be sure to check that and we will see if any further investigation is needed.

![x](/images/10.png)

还需要查看 `oauth_clients` 表的 `password_client` 是否为 1
