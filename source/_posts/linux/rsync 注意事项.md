---
title: rsync 注意事项.md
date: 2019-08-17 16:09:00
tags: [linux]
---

* rsync 的时候 --include --exclude 不会使用绝对路径，全都是相对于 rsync 源目录的路径

* --include 需要在 --exclude 前面

* include 和 exclude 的路径前加 "/" 指的是相对 rsync 源目录路径的目录，如果不加，所有子目录下匹配上该路径的都会被匹配

* 例子:
    * `rsync /data/c --include="/e/" --exclude="e" root@127.0.0.1:/data/d`，/data/c/d/e 会被同步, /data/c/e 不会被同步
    * `rsync /data/c --exclude="e" root@127.0.0.1:/data/d`， /data/c/d/e 和 /data/c/e 不会被同步，所有 /data/c 下目录名为 e 的都不会被同步（包括子目录下的）
    * `rsync /data/c --include='/c/d/e/' --exclude="e" root@127.0.0.1:/data/d`， /data/c/d/e 会被同步，所有 /data/c 下其他目录名为 e 的都不会被同步（包括子目录下的）
