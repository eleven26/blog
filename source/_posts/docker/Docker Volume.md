---
title: Docker Volume
date: 2019-07-07 09:03:30
tags: Docker
---

> Docker Volume 使用场景：数据共享、数据容器、备份。

想要了解 Docker Volume，首先我们要知道 Docker 的文件系统是如何工作的。Docker 镜像是由多个文件系统（只读层）叠加而成。当我们启动一个容器的时候，Docker 会加载只读镜像层并在其上添加一个读写层。
如果运行中的容器修改了现有的一个已经存在的文件，那该文件会从读写层下面的只读层复制到读写层，该文件的只读版本仍然存在，只是已经被读写层中该文件的副本所隐藏。当删除 Docker 容器，并通过该镜像重新启动时，
之前的更改将会丢失。在 Docker 中，只读层及在顶部的读写层的组合被称为 Union File System（联合文件系统）。

为了能够保存（持久化）数据以及共享容器间的数据，Docker 提出了 Volume 的概念。简单来说，Volume 就是目录或者文件，它可以绕过默认的联合文件系统，而以正常的文件或者目录的形式存在于宿主机上。

我们可以通过两种方式来初始化 Volume，这两种方式有些细小而又重要的区别。我们可以在运行时使用 -v 来声明 Volume:

```bash
docker run -it --name container-test -h CONTAINER -v /data debian /bin/bash
root@CONTAINER:/# ls /data
root@CONTAINER:/#
```

上面的命令会将 /data 挂载到容器中，并绕过联合文件系统，我们可以在主机上直接操作该目录。任何在该镜像 /data 路径的文件将会被复制到 Volume。我们可以使用 `docker inspect` 命令找到 Volume 在主机上的存储位置：

```bash
docker inspect -f {{.Volumes}} container-test
```

你会看到类似的输出:

```bash
map[/data:/var/lib/docker/vfs/dir/.....]
```

这说明 Docker 把 /var/lib/docker 下的某个目录挂载到了容器内的 /data 目录下。

这时如果我们在宿主机上的 /var/lib/docker 下对应目录新增一个文件，在容器内的 /data 就会看到一样的文件。

只要将宿主机的目录挂载到容器的目录上，那改变就会立即生效。我们可以在 Dockerfile 中通过 VOLUME 指令来达到相同的目的：

```dockerfile
FROM debian:wheezy
VOLUME /data
```

但还有另一件只有 -v 参数能够做到而 Dockerfile 是做不到的事情就是在容器上挂载指定的主机目录。例如：

```bash
docker run -v /home/tony/data:/data debian ls /data
```

该命令将挂载主机的 /home/tony/data 目录到容器内的 /data 目录上。任何在 /home/tony/data 目录的文件都会出现在容器内。这对于在主机和容器之间共享文件是非常有帮助的，例如挂载需要编译的源代码。
为了保证可移植性（并不少所有系统的宿主机目录都是可以用的），挂载主机目录不需要从 Dockerfile 指定。当使用 -v 参数时，镜像目录下的任何文件都不会被复制到 Volume 中。（Volume 会复制到镜像目录，镜像不会复制到卷）



## 数据共享

如果要授权一个容器访问另一个容器的 Volume，我们可以使用 --volumes-from 参数来执行 docker run。

```bash
docker run -it -h NEWCONTAINER --volumes-from container-test debian /bin/bash
```

值得注意的是不管 container-test 是否运行，它都会起作用。只要有容器连接 Volume，它就不会被删除。


## 数据容器

常见的使用场景是使用纯数据容器来持久化数据库、配置文件或者数据文件等。例如：

```bash
docker run --name dbdata postgres echo "Data-only container for postgres"
```

该命令将会创建一个已经包含在 Dockerfile 里定义过 Volume 的 postgres 镜像，运行 echo 命令然后退出。
当我们运行 `docker ps` 命令时，echo 可以帮助我们识别某镜像的用途。我们可以用 --volumes-from 命令来识别其他容器的 Volume:

```bash
docker run -d --volumes-from dbdata --name db1 postgres
```

使用数据容器的两个注意点：

* 不要运行数据容器，这纯粹是在浪费资源

* 不要为了数据容器而使用 "最小的镜像"，如 busybox 或 scratch，只使用数据库镜像本身就可以来。你已经拥有该镜像，所以并不需要占用额外的空间。


## 备份

如果你在用数据容器，那做备份是相当容易的：

```bash
$ docker run --rm --volumes-from dbdata -v $(pwd):/backup debian tar cvf /backup/backup.tar /var/lib/postgresql/data
```

该示例应该会将 Volume 里所有的东西压缩为一个 tar 包（官方的 postgres Dockerfile 在 /var/lib/postgres/data 目录下定义了一个 Volume）


## 权限与许可

通常你需要设置 Volume 的权限或者为 Volume 初始化一些默认数据或者配置文件。要注意的关键点是，在 Dockerfile 的 VOLUME 指令后的任何东西都不能改变该 Volume，比如：

```dockerfile
FROM debian:wheezy
RUN useradd foo
VOLUME /data
RUN touch /data/x
RUN chown -R foo:foo /data
```

该 Dockerfile 不能按预期那样运行，我们本来系统 `touch` 命令在镜像的文件系统上运行，但是实际上它是在一个临时容器的 Volume 上运行。如下所示：

```dockerfile
FROM debian:wheezy
RUN useradd foo
RUN mkdir /data && touch /data/x
RUN chown -R foo:foo /data
VOLUME /data
```

Docker 可以将镜像中 Volume 下的文件挂载到 Volume 下，并设置正确的权限。如果你指定 Volume 的主机目录将不会出现这种情况。

如果你没有通过 RUN 指令设置权限，那么你就需要在容器启动时使用 CMD 或 ENTRYPOINT 指令来执行。
（CMD 指令用于指定一个容器启动时要运行的命令，与 RUN 类似，只是 RUN 是镜像在构建时要运的命令。）


## 删除 Volumes

这个功能可能会更加重要，如果你已经使用 docker rm 来删除你的容器，那可能有很多的孤立的 Volume 仍在占用着空间。

Volume 只有在下列情况下才能被删除：

* 该容器是用 `docker rm -v` 命令来删除的（-v 是必不可少的）

* `docker run` 中使用了 `--rm` 参数

即使用以上两种命令，也只能没有被容器连接的 Volume。连接到用户指定主机目录的 Volume 永远不会被 docker 删除。

除非你已经很小心的，总是像这样来运行容器，否则你将在 /var/lib/docker/vfs/dir 目录下得到一些僵尸文件和目录，并且还不容器说出它们到底代表什么。


