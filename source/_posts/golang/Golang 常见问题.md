---
title: Golang 常见问题
date: 2020-03-29 09:11:30
tags: [Go]
---

1. 国内代理

```
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
```

[https://github.com/goproxy/goproxy.cn/blob/master/README.zh-CN.md](https://github.com/goproxy/goproxy.cn/blob/master/README.zh-CN.md)


2. 获取一个目录下的所有依赖

旧项目没有 go modules，需要使用这个命令

```
go get -d ./...
```

`...` 表示是递归地获取所有目录。

[how-to-get-all-dependency-files-for-a-program-using-golang](https://stackoverflow.com/questions/32758235/how-to-get-all-dependency-files-for-a-program-using-golang/43107943)

