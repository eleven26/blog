---
title: mac、homestead mongodb PHP 扩展安装
date: 2020-02-13 08:48:00
tags: [PHP, homestead, mongodb]
---

下载源码(下面指定了版本)

```
sudo rm -rf /tmp/mongo-php-driver /usr/src/mongo-php-driver
git clone -c advice.detachedHead=false -q -b '1.6.1' --single-branch https://github.com/mongodb/mongo-php-driver.git /tmp/mongo-php-driver
sudo mv /tmp/mongo-php-driver /usr/src/mongo-php-driver
cd /usr/src/mongo-php-driver
git submodule -q update --init
```

编译安装

```
phpize && ./configure && make && make install
```
