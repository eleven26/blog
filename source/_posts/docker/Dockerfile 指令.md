---
title: Dockerfile 指令
date: 2019-06-27 21:27:30
tags: Docker
---

### CMD

CMD 指令用于指定一个容器启动时需要运行的命令。这有点类似于 RUN 指令，只是 RUN 指令是指定镜像被构建时要运行的命令，而 CMD 是指定容器被启动时要运行的命令。这和使用 `docker run` 命令启动容器时指定要运行的命令非常类似。

```bash
sudo docker run -i -t jamtur01/static_web /bin/true
```

我们可以认为上面的命令和在 Dockerfile 中使用下面的指令是等效的。

```bash
CMD [ "/bin/true" ]
```

当然也可以为要运行的命令指定参数：

```bash
CMD [ "/bin/bash", "-l" ]
```

这里我们将 "-l" 标志传递给了 /bin/bash 命令。

需要注意的是，要运行的命令是存放在一个数组结构中。这将告诉 Docker 按指定的原样来运行该命令。当然也可以不使用数组而是指定 CMD 指令，这时候 Docker 会在指定的命令前加上 /bin/sh -c。这在执行该命令的时候可能会导致意料之外的行为，所以 Docker 推荐一直使用以数组语法来设置要执行的命令。

最后，还需牢记，使用 `docker run` 命令可以覆盖 CMD 指令。如果我们在 Dockerfile 里指定了 CMD 指令，而同时在 `docker run` 命令中也指定了要运行的命令，命令行中指定的命令会覆盖 Dockerfile 中的 CMD 指令。

假设我们在 Dockerfile 文件中有如下指令：

```bash
CMD [ "/bin/bash" ]
```

可以使用 `docker build` 命令构建一个新镜像（假设镜像名为 jamtur01/test），并基于此镜像启动一个容器。

```bash
sudo docker run -i -t jamtur01/test
```

在 `docker run` 命令的末尾我们并未指定要运行什么命令。实际上，Docker 使用了 CMD 指令中指定的命令。

覆盖本地命令：

```bash
sudo docker run -i -t jamtur01/test /bin/ps
```

可以看到，在这里我们指定了想要运行的命令 /bin/ps，该命令会列出所有正在运行的进程。在这个例子中，容器并没有启动 shell，而是通过命令行参数覆盖了 CMD 指令中指定的命令，容器运行后列出了正在运行的进程的列表。

#### 注意事项

在 Dockerfile 中只能指定一条 CMD 指令。如果指定了多条 CMD 指令，也只有一条 CMD 指令会被使用。如果想在启动容器时运行多个进程或者多条命令，可以考虑使用类似 Supervisor 这样的服务管理工具。


### ENTRYPOINT

ENTRYPOINT 指令与 CMD 指令非常相似。有时候，我们希望容器按照我们想象的那样去工作，这时候 CMD 指令就不太合适了。而 ENTRYPOINT 指令提供的命令则不容易在启动容器时被覆盖。实际上，`docker run` 命令中指定的任何参数都会被当作参数再次传递给 ENTRYPOINT 指令中指定的命令。

```dockerfile
ENTRYPOINT [ "/usr/sbin/nginx" ]
```

类似 CMD 指令，我们也可以在该指令中通过数组的方式为命令指定相应的参数：

```dockerfile
ENTRYPOINT [ "/usr/sbin/nginx", "-g", "daemon off;" ]
```

和之前提到的 CMD 指令一样，我们通过给 ENTRYPOINT 传入数组的方式来避免在命令前加入 `/bin/sh -c` 带来的各种问题。

现在我们重新构建我们的镜像，并将 ENTRYPOINT 设置为 `ENTRYPOINT ["/usr/sbin/nginx"]`

重新构建镜像：

```bash
sudo docker build -t="jamtur01/static_web" .
``` 

然后，我们从 jamtur01/static_web 镜像启动一个新容器。

使用 `docker run` 命令启动包含 ENTRYPOINT 指令的容器：

```bash
sudo docker run -t -i jamtur01/static_web -g "daemon off;"
``` 

在上面的命令中，我们指定了 `-g "daemon off;"` 参数，这个参数会传递给用 ENTRYPOINT 指令的命令，在这里该命令为 `/usr/sbin/nginx -f "daemon off;"`。改命令会以前台运行的方式启动 Nginx 守护进程，此时这个容器就会作为一台 Web 服务器来运行。

我们也可以组合使用 ENTRYPOINT 和 CMD 指令来完成一些巧妙的工作。比如：

```dockerfile
ENTRYPOINT [ "/usr/sbin/nginx" ]
CMD [ "-h" ]
```

此时我们启动一个容器，任何在命令行中指定的参数都会被传递给 Nginx 守护进程。比如，我们可以指定 -g "daemon off;" 参数让 Nginx 守护进程以前台方式运行。如果在启动容器时不指定任何参数，则在 CMD 指令中指定的 -h 参数会被传递给 nginx 守护进程，即 nginx 服务器会以 `/usr/sbin/nginx -h` 的方式启动，该命令用来显示 nginx 的帮助信息。

这使我们可以构建一个镜像，该镜像既可以运行一个默认的命令，同时它也支持通过 `docker run` 命令行为该命令指定可覆盖的选项或者标志。

##### 提示

如果确实需要，用户也可以在运行时通过 `docker run` 的 --entrypoint 标志覆盖 ENTRYPOINT 指令。


### WORKDIR

WORKDIR 指令用来在从镜像创建一个新容器时，在容器内部设置一个工作目录，ENTRYPOINT 和 / 或 CMD 指令指定的程序会在这个目录下执行。

 
