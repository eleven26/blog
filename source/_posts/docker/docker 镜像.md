---
title: Docker 镜像
date: 2019-06-16 18:37:02
tags: Docker
---

#### 什么是 Docker 镜像

Docker 镜像是由文件系统叠加而成。最底端是一个引导文件系统，即 bootfs，这很像典型的 Linux/Unix 的引导文件系统。Docker 用户几乎永远不会和引导文件系统有什么交互。实际上，当一个容器启动后，它会被移到内存中，而引导文件系统会被卸载（umount），以留出更多的内存供 initrd 磁盘镜像使用。

Docker 看起来还很像一个典型的 Linux 虚拟化栈。实际上，Docker 镜像的第二层是 root 文件系统 rootfs，它位于引导文件系统之上。rootfs 可以是一种或多种操作系统（如 Debian 或者 Ubuntu 文件系统）。

在传统的 Linux 引导过程中。root 文件系统会最先以只读的方式加载，当引导结束并完成了完整性检查后，它才会被切换为读写模式。

但是在 Docker 里，root 文件系统永远只能是只读状态，并且 Docker 利用联合加载 （union mount）技术又会在 root 文件系统层上加载更多的只读文件系统。联合加载会将各层文件系统叠加到一起，这样最终的文件系统会包含所有底层的文件和目录。

Docker 将这样的文件系统称为镜像。一个镜像可以放到另一个镜像的顶部。位于下面的镜像被称为父镜像（parent image），可以以此类推，直到镜像栈的最底部，最底部的镜像称为基础镜像（base image）。最后，当一个镜像启动容器时，Docker 会在该镜像的最顶层加载一个读写文件系统。我们想在 Docker 中运行的程序就是在这个读写层中执行的。

当 Docker 第一次启动一个容器时，初始的读写层是空的。当文件系统发生变化时，这些变化都会应用到这一层上。比如，如果想修改一个文件，这个文件会先从读写层下面的只读层复制到该读写层。该文件的只读版本依然存在，但是已被读写层中的该文件副本所隐藏。

通常这种机制被称为写时复制（copy on write），这也是使 Docker 如此强大的技术之一。每个只读镜像层都是只读的，并且以后永远不会变化。当创建一个新容器时，Docker 会创建出一个镜像栈，并在栈的最顶端添加一个读写层。这个读写层再加上其下面的镜像层以及一些配置数据，就构成了一个容器。


#### 列出镜像

`docker images`

本地镜像都保存在 Docker 宿主机的 `/var/lib/docker` 目录下。每个镜像都保存在 Docker 所采用的存储驱动目录下面，如 aufs 或者 devicemanager。也可以在 `/var/lib/docker/containers` 目录下面看到所有的容器。

镜像从仓库下载下来。镜像保存在仓库中，而仓库存在于 Registry 中。

每个镜像仓库都可以存放很多镜像（比如，ubuntu 仓库包含了 Ubuntu12.04 、 12.10 和 14.04 的镜像）。

`sudo docker pull ubuntu:12.04`

再次运行 `docker images` 可以看到最先拉取的 12.04 镜像。

为了区分同一个仓库中的不同镜像，Docker 提供了一种称为标签（tag）的功能。每个镜像在列出来时都带有一个标签，如 12.04 、 12.10。每个标签对组成特定镜像的镜像层进行标记。这种机制使得同一个仓库中可以存储多个镜像。

我们可以通过在仓库名后面加上一个冒号和标签名来指定该仓库中某一镜像。

* 运行一个带标签的 Docker 镜像

```bash
sudo docker run -t -i --name new_container ubuntu:12.04 /bin/bash
```

这个例子会从镜像 ubuntu12.04 启动一个容器，而这个镜像的操作系统则是 Ubuntu12.04。

在构建容器时指定仓库的标签也是一个很好的习惯，这样便可以准确地指定容器来源于哪里。不同标签的镜像会有不同，比如 Ubuntu12.04 和 14.04 就不一样，指定镜像的标签会让我们准确知道自己使用的是 ubuntu:12.04，这样我们就可以准确知道自己在干什么。

Docker Hub 中有两种类型的仓库：用户仓库（user repository）和顶层仓库（top-level repository）。用户仓库的镜像都是由 Docker 用户创建的，而顶层仓库则是由 Docker 内部的人来管理的。

用户仓库的命名由用户名和仓库名两部分组成的，如 jamtur01/puppet。

* 用户名：jamtur01

* 仓库名：puppet

与之相对，顶层仓库只包含仓库名部分，如 ubuntu 仓库。顶层仓库由 Docker 公司和由选定的能提供优质基础镜像的厂商管理，用于可以基于这些基础镜像构建自己的镜像。


#### 拉取镜像

用 `docker run` 命令从镜像启动一个容器时，如果该镜像不在本地，Docker 会先从 Docker Hub 下载该镜像。如果没有指定具体的镜像标签，那么 Docker 会自动下载 latest 标签的镜像。

```bash
sudo docker run -t -i --name next_container ubuntu /bin/bash
```

这个命令，如果本地么没有 ubuntu:latest 这个镜像，会从远程拉取到本地。使用 `docker pull` 命令可以节省从一个新镜像启动一个容器所需的时间。

```bash
sudo docker pull fedora:20
```

可以使用 `docker images` 命令看到这个新镜像已经下载到本地 Docker 宿主机上了。不过这次我们希望能在镜像列表中只看到 fedora 镜像的内容。这可以通过在 `docker iamges` 命令后面指定镜像名来实现。

```bash
sudo docker images fedora
```


#### 查找镜像

我们也可以通过 `docker search` 命令来查找所有 Docker hub 上公共的可用镜像

```bash
sudo docker search puppet
```

上面的命令在 Docker Hub 上查找了所有带有 puppet 的镜像。这条命令去完成镜像查找工作。

我们从上面的结果中选取一个 pull 到本地：

```bash
sudo docker pull puppet/puppetdb
```

这条命令将会下载 puppet/puppetdb 镜像到本地。接着就可以使用这个镜像构建一个容器了。下面就用 `docker run` 命令来构建一个容器。

```bash
sudo docker run -t -i puppet/puppetdb /bin/bash
```


#### 构建镜像

前面我们已经看到了如何拉取已经构建好的带有定制内容的 Docker 镜像，那么我们如何修改自己的镜像，并且更新和管理这些镜像呢？

构建 Docker 镜像有以下两种方法，

* 使用 `docker commit` 命令。

* 使用 `docker build` 命令 和 `Dockerfile` 文件。

现在我们并不推荐使用 `docker commit` 命令，而应该使用更灵活、更强大的 `Dockerfile` 来构建 Docker 镜像。

##### 用 Docker 的 commit 命令创建镜像

创建 Docker 镜像的第一种方法是使用 `docker commit` 命令。可以将此想象为我们在往版本控制系统提交变更。我们先创建一个容器，并在容器里作出修改，就像修改代码一样，最后再将修改提交为一个新镜像。

* 创建一个要进行修改的定制容器

```bash
sudo docker run -t -i ubuntu /bin/bash
```

接下来，在容器中安装 Apache

```bash
apt-get -yqq update
apt-get -y install apache2
```

我们启动了一个容器，并在里面安装了 Apache。我们会将这个容器作为一个 Web 服务器来运行，所以我们想把它的当前状态保存下来。这样就不必每次都创建一个新容器并再次在里面安装 Apache 了。为了完成此项工作，需要先使用 exit 命令从容器里退出，之后再运行 `docker commit` 命令。

* 提交定制容器

```bash
sudo docker commit 4aab3ceksld jamtur01/apache2
```

