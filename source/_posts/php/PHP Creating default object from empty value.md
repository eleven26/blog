---
title: PHP Creating default object from empty value
date: 2020-03-26 09:22:00
tags: [PHP]
---

```
$res = NULL;
$res->success = false; // Warning: Creating default object from empty value
```

产生原因是给一个值为 null 的变量设置属性。
