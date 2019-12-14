---
title: git clone fatal Unable to read current directory
date: 2019-10-23 09:19:30
tags: Git
---

执行命令的目录: `/Volumes/data/jenkins/workspace/qa3`

报错

```
Cloning into '/home/web/deployer/Foundation/qa3/default/releases/20191022203935'...
fatal: Unable to read current working directory: Permission denied
fatal: index-pack failed
```

解决方法: 进入到对应目录执行 clone

`cd /home/web/deployer/Foundation/qa3/default/releases && git clone xxx 20191022203935`
