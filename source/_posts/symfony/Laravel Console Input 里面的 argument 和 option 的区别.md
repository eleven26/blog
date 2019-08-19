---
title: Laravel Console Input 里面的 argument 和 option 的区别
date: 2019-07-11 17:06:00
tags: [Symfony, Laravel]
---

我们在使用 Laravel 的 Command 的时候会发现，里面有两个方法 `argument`、`option`，可是这两个方法有什么区别呢？

### Argument

argument 是空格分隔的多个字符串，并且是有序的。

比如:

```bash
php test.php abc
```

在 php 里面，如果我们 `var_dump($argv)`，就可以看到里面有 `test.php` 和 `abc`，这些就是 argument 了，并且他们是有顺序的。


### option

option 是以 -- 开头的 argument，但是 Laravel 里面会把其当作是 option 而不是 argument。

比如:

```bash
php test.php abc --name=php
```

这里的 `test.php` 和 `abc` 是 argument，而 `name` 是一个 option。


### 总结

* argument 前面没有 --，option 前面有 --

* argument 是有序的，如果顺序乱了，会导致 `$this->argument` 拿到的 argument 是错误的


[参考文档](https://symfony.com/doc/current/console/input.html)

