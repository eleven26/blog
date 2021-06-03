---
title: linux 共享库找不到的问题
date: 2021-06-03 10:52:00
tags: [linux]
---

是这样的，在服务器上的 php 需要 zip 库，但我自己手动编译了一个 zip 库，安装在了 `/usr/local/lib64` 下。但 php 编译 zip 扩展的时候提示找不到 libzip。

## 解决方法

将 `/usr/local/lib64` 也加入到系统共享库的搜索路径，我们可以通过 `ldconfig` 

```shell script
echo "/usr/local/lib64" > /etc/ld.so.conf.d/local.conf
ldconfig
```

然后重新 configure。
