---
title: Golang 获取一个目录下所有的依赖
date: 2020-03-29 09:11:30
tags: [Go]
---

可以使用 `go get -d ./...` 来下载某一个项目下所有可以使用 go get 下载的依赖。

`...` 表示是递归地获取所有目录。


参考链接：

[how-to-get-all-dependency-files-for-a-program-using-golang](https://stackoverflow.com/questions/32758235/how-to-get-all-dependency-files-for-a-program-using-golang/43107943)
