---
title: Docker Volume（二）
date: 2019-07-13 09:03:30
tags: Docker
---

很多人都对 Volume 有一个误解，他们认为 Volume 是为了持久化。如此想法是因为他们觉得容器不能持久化，所以 Volume 应该是为了满足这个需求设计的。其实容器会一直存在，除非你删除它们：

这可能来自于容器不是持久的想法，这样确实是不对的。容器是持久的，直到你删除他们，并且你只能这样做：

```bash
docker rm my_container
```

如果你没有执行此命令，那么你的容器会一直存在，依旧可以启动、停止等。如果你找不到你的容器，可以运行此命令：

```bash
docker ps -a
```

`docker ps` 只能显示正在运行的容器，但是容器也会处于停止状态，这种情况下，上面的命令会显示所有的容器，无论它们处于什么状态。

综上，再次声明：Volume 并不是为了持久化。


### 什么是 Volume

Volume 可以将容器以及容器产生的数据分离开来，这样，当你使用 `docker rm my_container` 删除容器时，不会影响相关的数据。

Volume 可以使用以下两种方式创建：

* 在 Dockerfile 中指定 `VOLUME /some/dir`

* 执行 `docker run -v /some/dir` 命令来指定

无论哪种方式都是做了同样的事情。它们告诉 Docker 在主机上创建一个目录（默认情况下是在 `/var/lib/docker` 下），然后将其挂载到指定的路径（例子中是 `/some/dir`）。当删除使用该 Volume 的容器时，Volume 本身不会受到影响，它可以一直存在下去。

如果在容器中不存在指定的路径，那么该目录将会被自动创建。

你可以告诉 Docker 同时删除容器和其 Volume：

```bash
docker rm -v my_container
```

有时候，你想在容器中使用主机上的某个目录，你可以通过其它的参数来指定：

```bash
docker run -v /host/path:/some/path
```

这明确地告诉 Docker 使用指定的主机路径来代替 Docker 自己创建的根路径并挂载到容器内指定的路径（以上例子为 `/some/path`）。需要注意的是，这种方式同样支持文件。在 Docker 术语中，这通常被称为 `bind-mount`（虽然技术层面上是这样讲的，但是实际的感觉是所有的 Volume 都是 bind-mounts 的）。
如果主机上的路径不存在，目录将自动在给定的路径中创建。

对待 bind-mounts Volume 和一个正常的 Volume 有一点不同，它不会修改主机上那些非 Docker 创建的东西：

* 一个正常的 Volume，Docker 会自动将指定 Volume 路径（如上面的示例 `/some/path`）上的数据复制到由 Docker 创建的新目录下，如果是 "bind-mount"。Volume 就不会这样做。（这样做会将主机上的目录复制到容器）

* 当你执行 `docker rm -v my_container` 命令时，"bind-mount" 类型的 Volume 不会被删除

