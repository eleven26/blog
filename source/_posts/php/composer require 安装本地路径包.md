---
title: composer require 安装本地路径包
date: 2019-03-21 13:53:30
tags: Composer
---

1. 在 composer.json 添加下面的配置

```
"repositories": [
    {
        "type": "path",
        "url": "/full/or/relative/path/to/development/package"
    }
]
```

2. 执行 `require` 操作

```
composer require "vendorname/packagename @dev"
```

> @dev 和包名中间的空格要有


[查看原文](https://stackoverflow.com/questions/29994088/composer-require-local-package)
