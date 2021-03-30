---
title: Laravel  Creating default object from empty value 报错原因
date: 2021-03-30 10:30:00
tags: [Laravel, PHP]
---

当我们尝试设置一个 `null` 变量的属性的时候，会报这个错：

```
$a = null;
$a->b = 1; // Creating default object from empty value
```

其实除了 `null`，值为 空字符串 或者 false 的变量，如果想设置其属性的时候也会报这个错：

```
$a = ''; // 或者 $a = false;
$a->b = 1; // Creating default object from empty value
```
