---
title: Laravel Schema 查看索引是否存在
date: 2019-03-21 13:53:00
tags: [Laravel, PHP]
---

```php
Schema::connection('')->table($tableName, function (Blueprint $table) {
    $sm = Schema::getConnection()->getDoctrineSchemaManager();
    $indexesFound = $sm->listTableIndexes('table');

    if(array_key_exists("index_name", $indexesFound))
        $table->dropUnique("index_name");
});
```
