---
title: linux 执行文件换行符识别不了处理
date: 2021-07-10 15:31:00
tags: [linux]
---

在 linux 下执行 shell 脚本的时候，有可能会报错 `line 2: $'\r': command not found`

出现这种错误的原因是，shell 识别不了 windows 的换行符 `\r\n`

## Ubuntu 解决方法

```shell script
sudo apt-get install tofrodos
fromdos 文件名
```

## Centos 解决方法

```shell script
yum -y install dos2unix
dos2unix 文件名
```
