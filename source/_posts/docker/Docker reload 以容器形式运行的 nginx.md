---
title: Docker reload 以容器形式运行的 nginx
date: 2019-10-24 09:06:30
tags: Docker
---

方法: 使用 `docker [container] exec`，container 可以省略，可以直接使用 `docker exec`

在使用 docker 容器的时候，我们总会想看看容器内部长什么样子: 我们可以使用 `docker exec` 命令。

现在，我们想通过 `nginx -s reload` 来重启 `nginx` 容器，可以使用下面的命令：

```shell script
docker exec nginx nginx -s reload
```

上面第一个 `nginx` 是容器名称，`nginx -s reload` 是进入到容器后运行的命令。

如果我们想以交互的方式进入容器（类似 `docker run`）:

```shell script
docker exec -it centos bash
```
