---
title: Python3 SSL 支持
date: 2019-12-09 17:35:00
tags: [Python]
---

Python3 开始不再支持 `./configure --with-ssl` 的方式引入 ssl，它会在编译的时候
自动检查系统是否有 ssl 的库，有则自动加上。前提是我们需要在系统安装了 `openssl-devel` 或者 `libssl-dev` 类似的库（取决于操作系统）。
