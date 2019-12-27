---
title: windows go get 加速
date: 2019-12-27 22:37:30
tags: [Go]
---

前提，有 ss，打开 git bash，运行：

```
export http_proxy=http://127.0.0.1:1080;export https_proxy=http://127.0.0.1:1080;
```

当然这里端口可能不一样，需要根据实际端口修改。

然后，再运行 `go get xxx` （在同一个终端）

如果想打开 git bash 的时候默认使用 http 代理，可以在用户主目录添加一个 `.bashrc` 文件，里面就写：

`export http_proxy=http://127.0.0.1:1080;export https_proxy=http://127.0.0.1:1080;`
