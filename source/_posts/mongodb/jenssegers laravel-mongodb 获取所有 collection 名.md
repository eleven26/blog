---
title: jenssegers laravel-mongodb 获取所有 collection 名
date: 2019-08-12 17:52:00
tags: [MongoDB, Laravel]
---

参考链接: [List all collections](https://github.com/jenssegers/laravel-mongodb/issues/1464#issuecomment-458698014)

```php
foreach (\DB::connection('xxx')->getMongoDB()->listCollections() as $collection) {
    echo $collection->getName() . PHP_EOL;
}
```

