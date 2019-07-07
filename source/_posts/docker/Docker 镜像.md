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

`4aab3ceksld` 是容器 ID，`jamtur01/apache2` 是镜像仓库和镜像名。需要注意的是，`docker commit` 提交的只是创建容器的镜像与容器当前状态之间有差异的部分，这使得该更新非常轻量。

* 检查新创建的镜像

```bash
sudo docker images jamtur01/apache2
```

也可以在提交镜像时指定更多的数据（包括标签）来详细描述所做的修改。

```bash
sudo docker commit -m"A new custom image" -a"James Turnbull" 4aab3ceksld jamtur01/apache2:webserver
```

在这条命令里，我们指定了更多的信息选项。首先 -m 选项用来指定新创建的镜像的提交信息。同时还指定了 -a 选项，用来列出该镜像的作者。接着指定了想要提交的容器的 ID。最后的 jamtur01/apache2 指定了镜像的用户名和仓库名，并为该镜像增加了一个 webserver 标签。

可以用 `docker inspect` 命令来查看新创建的镜像的详细信息：

```bash
sudo docker inspect jamtur01/apache2:webserver
```

如果想从刚创建的新镜像运行一个容器，可以使用 `docker run` 命令。

```bash
sudo docker run -t -i jamtur01/apache2:webserver /bin/bash
```

##### 用 Dockerfile 构建镜像

```dockerfile
# Version: 0.0.1
FROM ubuntu:14.04
MAINTAINER James Turnbull "james@example.com"
RUN apt-get update && apt-get install -y nginx
RUN echo 'Hi, I am in your container' > /usr/share/nginx/html/index.html
EXPOSE 80 
```

该 Dockerfile 由一系列指令和参数组成。每条指令，如 FROM，都必须为大写字母，且后面要跟随一个参数：FROM ubuntu:14.04。Dockerfile 中的指令会会按顺序从上到下执行，所以应该根据需要合理安排指令的顺序。

每条指令都会创建一个新的镜像层并对镜像进行提交。Docker 大体上按照如下流程执行 Dockerfile 中的指令。

* Docker 从基础镜像运行一个容器

* 执行一条指令，对容器作出修改

* 执行类似 `docker commit` 的操作，提交一个新的镜像层

* Docker 再基于刚提交的镜像运行一个新容器

* 执行 Dockerfile 中的下一条指令，直到所有指令执行完毕。

每个 Dockerfile 的第一条指令必须是 FROM。FROM 指令指定一个已经存在的镜像，后续指令都将基于该镜像进行，这个镜像被称为基础镜像（base image）。

每条 RUN 指令都会创建一个新的镜像层，如果该指令执行成功，就会将此镜像层提交，之后继续执行 Dockerfile 中的下一条指令。

默认情况下，RUN 指令会在 shell 里使用命令包装器 /bin/sh -c 来执行。如果是在一个不支持 shell 平台上运行或者不希望在 shell 中运行（比如避免 shell 字符串篡改），也可以使用 exec 格式的 RUN 指令。

```bash
RUN [ "apt-get", " install", "-y", "nginx" ]
```

在这种方式中，我们使用一个数组来指定要运行的命令和传递给该命令的每个参数。

接着设置了 EXPOSE 指令，这条指令告诉 Docker 容器该容器内的应用程序将会使用容器的指定端口。这并不意味着可以自动访问容器运行中服务的端口（这里是 80）。出于安全的原因，Docker 并不会自动打开该端口，而是需要用户在使用 `docker run` 运行容器时来指定需要打开哪些端口。

如果没有指定任何标签，Docker 将会自动为镜像设置一个 latest 标签。

```bash
sudo docker build -t="jamtur01/static_web:v1" .
```

上面的 `.` 告诉 Docker 到本地目录去找 Dockerfile 文件。也可以指定一个 Git 仓库的源地址来指定 Dockerfile 的位置。

也可以通过 -f 标志指定一个区别于标准 Dockerfile 的构建源的位置。

从新镜像启动容器：

```bash
sudo docker run -d -p 80:80 --name static_web jamtur01/static_web:v1 nginx -g "daemon off;"
```

* -d 选项，告诉 Docker 以分离 (detached) 的方式在后台运行

* -p 标志，用来控制 Docker 在运行时应该公开哪些网络端口给外部（宿主机）。

运行一个容器时，Docker 可以通过两种方法来在宿主机上分配端口。

* Docker 可以在宿主机上随机选择一个位于 32768~61000 的一个比较大的端口号来映射到容器的 80 端口上。

* 可以在 Docker 宿主机中指定一个具体的端口来映射到容器中的 80 端口上。

可以使用命令 `docker ps -l` 来查看容器的端口分配情况。

也可以通过 `docker port` 来查看容器的端口映射情况。

```bash
sudo docker port static_web 80
```

绑定不同的端口：

```bash
sudo docker run -d -p 8080:80 --name static_web jamtur01/static_web:v1 nginx -g "daemon off;"
```

这条命令会将容器中的 80 端口绑定到宿主机的 8080 端口上。

绑定到特定的网络接口：

```bash
sudo docker run -d -p 127.0.0.1:80:80 --name static_web jamtur01/static_web:v1 nginx -g "daemon off;"
```

这条命令会将容器内的 80 端口绑定到本地宿主机的 127.0.0.1 这个 IP 的 80 端口上。我们也可以使用类似的方式将容器内的 80 端口绑定到一个宿主机的随机端口上：

```bash
sudo docker run -d -p 127.0.0.1::80 --name static_web jamtur01/static_web:v1 nginx -g "daemon off;"
```

这时我么可以使用 `docker inspect` 或者 `docker port` 命令来查看容器内的 80 端口具体被绑定到了宿主机哪个端口上。

Docker 还提供了一个更简单的方式，即 -P 参数，该参数可以用来对外公开在 Dockerfile 中通过 EXPOSE 指令公开的所有端口。

```bash
sudo docker run -d -P --name static_web jamtur01/static_web nginx -g "daemon off;"
```

改命令会将容器内的 80 端口对本地宿主机公开，并且绑定到宿主机的一个随机端口上。

##### .dockerignore

如果在构建上下文目录下存在以 .dockerignore 命名的文件的话，那么该文件内容会被按行进行分割，每一行都是一条文件过滤匹配模式。这非常像 .gitignore 文件，该文件用来设置哪些文件不会被当作构建上下文的一部分，因此可以防止它们被上传到 Docker 守护进程中去。该文件中模式的匹配规则采用了 Go 语言中的 filepath。


##### Dockerfile 和构建缓存

由于每一步的构建过程都会将结果提交为镜像，所以 Docker 的构建镜像过程就显得非常聪明。我们在构建失败的时候可以使用之前某一步的构建缓存作为新的开始点。

然而，有些时候需要确保构建过程不会使用缓存。要想略过缓存功能，可以使用 `docker build` 的 `--no-cache` 标志。



