---
title: composer install 忽略平台依赖
date: 2020-02-28 09:48:00
tags: [PHP, Composer]
---

有时候我们在执行 `composer install` 的时候会遇到一些依赖不满足导致命令执行失败的情况，比如 php 版本不满足、扩展缺失等。如：

```
Your requirements could not be resolved to an installable set of packages.

  Problem 1
    - phpdocumentor/reflection-docblock 5.1.0 requires php ^7.2 -> your PHP version (7.1.14) does not satisfy that requirement.
    - phpdocumentor/reflection-docblock 5.1.0 requires php ^7.2 -> your PHP version (7.1.14) does not satisfy that requirement.
    - phpdocumentor/reflection-docblock 5.1.0 requires php ^7.2 -> your PHP version (7.1.14) does not satisfy that requirement.
    - Installation request for phpdocumentor/reflection-docblock (locked at 5.1.0) -> satisfiable by phpdocumentor/reflection-docblock[5.1.0].


Installation failed, reverting ./composer.json to its original content.
```

但有时候我们可能并不需要满足那个依赖也能满足我们项目的正常运行，这个时候我们可以选择忽略掉这些依赖不满足的情况继续执行 composer install。

这个时候，我们只需要加上参数 `--ignore-platform-reqs`

```
composer install --ignore-platform-reqs
```

加上这个参数之后，我们的 composer install 命令将会忽略掉依赖不满足的情况继续执行安装。
