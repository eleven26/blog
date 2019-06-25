---
title: Docker
date: 2019-06-16 18:37:02
tags: Docker
---

#### 命令

* 查看 docker 信息

`docker info`

* 运行第一个容器

`sudo docker run -i -t ubuntu /bin/bash`

命令详解

首先，我们告诉 Docker 执行 `docker run` 命令，并指定了 -i 和 -t 两个命令行参数。

-i 标志保证容器中 STDIN 是开启的，尽管我们并没有附着到容器中。

-t 告诉 Docker 为要创建的容器分配一个伪 tty 终端。这样，新创建的容器才能提供一个交互式 shell。

若要在命令行下创建一个我们能与之交互的容器，而不是一个运行后台服务的容器，则这两个参数已经是最基本的参数了。

接下来，我们告诉 Docker 基于什么镜像来创建容器，示例中使用的是 ubuntu 镜像。

最后我们告诉 Docker 在新容器中要运行什么命令，本例是 `/bin/bash`，当容器创建完毕之后，Docker 就会执行容器中的 `/bin/bash` 命令。

* 退出

`exit`，运行该命令的时候，容器会停止运行。只有在指定的 `/bin/bash` 命令处于运行状态的时候，我们的容器才会相应地处于运行状态。

一旦退出容器，`/bin/bash` 命令也就结束了，这时容器也随之停止了运行。

但是，这个时候，容器仍然是存在的，可以用 `docker ps -a` 命令查看当前系统中容器的列表。


* 查看容器进程 `docker ps`

执行 `docker ps` 命令时，只能看到当前正在运行的容器。如果指定 `-a` 标志的话，那么可以列出所有容器，包括正在运行的和已经停止的。

`docker ps -l`，列出最后一个运行的容器，无论其正在运行还是已经停止。


#### 容器别名

Docker 会为我们创建的每一个容器自动生成一个随机的名称。

我们可以使用 --name 标志来给容器指定一个名称。

`sudo docker run -i -t --name my_new_ubuntu ubuntu /bin/bash`

在很多 Docker 命令中，都可以用容器的名称来替代容器 ID。

容器名称必须是唯一的。如果要使用的容器名称已经存在，可以先用 `docker rm` 命令删除已有的同名容器后，再来创建新的容器。


#### 重新启动已经停止的容器

`sudo docker start my_new_ubuntu`

也可以通过容器 ID 来指定容器：

`sudo docker start 4aa9bef84953`

也可以使用 `docker restart` 命令来重新启动一个容器。

类似的，Docker 也提供了 `docker create` 命令来创建一个容器，但是并不运行它。


#### 附着到容器上

Docker 容器重新启动的时候，会沿用 `docker run` 命令时指定的参数来运行，因此我们的容器重新启动后会运行一个交互式会话 shell。

此外，也可以用 `docker attach` 命令重新附着到该容器会话上。

`sudo docker attach my_new_container`

通过容器 ID 重新附着到容器的会话上：

`sudo docker attach 4aa9bef84953`


#### 创建守护式容器

除了这些交互运行的容器（interactive container），也可以创建长期运行的容器。

守护式容器（daemonized container）没有交互式会话，非常适合运行应用程序和服务。

大多数时候我们都需要以守护式来运行我们的容器。

创建一个长期运行的容器：

`sudo docker run --name daemon_dave -d ubuntu /bin/bash -c "while true; do echo hello world; sleep 1; done"`

我们在上面的 `docker run` 命令使用了 -d 参数，因此 Docker 会将容器放到后台运行。

我们还在容器要运行的命令里使用了一个 while 循环，该循环会一直打印 hello world，直到容器或进程停止运行。

如果执行 `docker ps` 命令，可以看到一个正在运行的容器。


#### 查看容器日志

`sudo docker logs daemon_dave`

上述命令会输出最后几条日志并返回。我们可以在命令后使用 -f 参数来监控 Docker 的日志，这与 `tail -f` 命令非常相似。

显示时间戳：-t

`docker logs daemon_dave -f -t`


#### 查看容器内的进程

`sudo docker top daemon_dave`


#### Docker 统计信息

`sudo docker stats daemon_dave`

我们能看到一个守护式容器的列表，以及它们的 CPU、内存、网络I/O及存储I/O的性能和指标。


#### 在容器内部运行进程

可以通过 `docker exec` 命令在容器内部额外启动新进程。可以在容器内运行的进程有两种类型：后台任务和交互式任务。

后台任务在容器内运行且没有交互需求，而交互式任务则保持在前台运行。

运行守护进程

`sudo docker exec -d daemon_dave touch /etc/new_config_file`

-d 表明需要运行一个后台进程，-d 标志之后，指定的是要在内部执行这个命令的容器以及要执行的命令。

上面的例子会在 daemon_dave 容器内创建一个空文件，文件名为 `/etc/new_config/file`。

通过 `docker exec` 后台命令，可以在正在运行的容器中进行维护、监控及管理任务。

也可以使用 -u 指定进程所属用户。

运行交互式命令

`sudo docker exec -t -i daemon_dave /bin/bash`

和 `docker run` 同理。


#### 停止守护式容器

`sudo docker stop daemon_dave`

通过 ID 停止正在运行的容器

`sudo docker stop 4aa9bef84953`

`docker stop` 命令会向 Docker 容器进程发送 SIGTERM 信号。如果想快速停止某个容器，也可以使用 `docker kill` 命令来向容器进程发送 SIGKILL 信号。

要想查看已经停止的容器的状态，则可以使用 `docker ps` 命令。

还有一个很实用的命令 `docker ps -n x`，该命令会显示最后 x 个容器，不论这些容器正在运行还是已经停止。


#### 自动重启容器

如果由于某种错误而导致容器停止运行，还可以通过 --restart 标志，让 Docker 自动重新启动容器。

--restart 标志会检查容器的退出代码，并根据此来决定是否要重启容器。默认的行为 Docker 是不会重启容器的。

`sudo docker run --restart=always --name daemon_dave -d ubuntu /bin/bash -c "while true; do echo hello world; sleep 1; done""`

在本例中，--restart 标志被设置为 always。无论容器的退出代码是什么，Docker 都会自动重启该容器。

除了 always，还可以将这个标志设为 on-failures，这样，只有当前容器的退出代码为非 0 值的时候，才会自动重启。另外，on-failures 还接受一个可选的重启次数参数。

`--restart=on-failures:5`


#### 深入容器

`docker inspect`

`docker inspect` 命令会对容器进行详细的检查，然后返回其配置信息，包括名称、命令、网络配置以及很多有用的数据。

也可以使用 -f 或者 --format 标志来选定查看结果。

```
sudo docker inspect --format='{{ .State.Running }}' daemon_dave
```

返回容器运行状态

```
sudo docker inspect --format='{{ .NetworkSettings.IPAddress }}' daemon_dave
```

返回容器 IP 地址


#### 删除容器

`docker rm daemon_dave` 如果容器正在运行，可以使用 `docker rm -f daemon_dave` 来强制删除容器。

目前，还没有办法一次删除所有容器，不过可以通过下面的命令来实现：

```
sudo docker rm `sudo docker ps -a -q`
```

上面的 `docker ps` 命令会列出现有的全部容器，-a 标志代表列出所有容器，而 -q 标志则表示只需要返回容器的 ID 而不会返回容器的其他信息。

这样我们就得到了容器 ID 的列表，并传给了 docker rm 命令，从而达到删除所有容器的目的。


