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

重新构建镜像:

```bash
sudo docker build -t="jamtur01/static_web" .
```

然后，我们从 jamtur01/static_web 镜像启动一个新容器。

使用 `docker run` 命令启动包含 ENTRYPOINT 指令的容器:

```bash
sudo docker run -t -i jamtur01/static_web -g "daemon off;"
```

在上面的命令中，我们指定了 `-g "daemon off;"` 参数，这个参数会传递给用 ENTRYPOINT 指令的命令，在这里该命令为 `/usr/sbin/nginx -f "daemon off;"`。改命令会以前台运行的方式启动 Nginx 守护进程，此时这个容器就会作为一台 Web 服务器来运行。

我们也可以组合使用 ENTRYPOINT 和 CMD 指令来完成一些巧妙的工作。比如:

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

我们可以使用该指令为 Dockerfile 中后续的一系列指令设置工作目录，也可以为最终的容器设置工作目录。（可以使用多次，最后一个 WORKDIR 指定的目录为最终容器的工作目录。）

```dockerfile
WORKDIR /opt/webapp/db
RUN bundle install
WORKDIR /opt/webapp
ENTRYPOINT [ "rackup" ]
``` 

这里，我们将工作目录切换为 /opt/webapp/db 后运行了 bundle install 命令，之后又将工作目录设置为 /opt/webapp，最后设置了 ENTRYPOINT 来启动 rackup 命令。

可以通过 -w 标志在运行时覆盖工作目录，如下：

```bash
sudo docker run -ti -w /var/log ubuntu pwd
```

该命令会将容器内的工作目录设置为 /var/log


### ENV

ENV 用来在镜像构建过程中设置环境变量。

在 Dockerfile 文件中设置环境变量：

```dockerfile
ENV RVM_PATH /home/rvm/
```

这个新的环境变量可以在后续的任何 RUN 指令中使用，这就如同在命令前面指定了环境变量前缀一样。

```dockerfile
RUN gem install unicorn
```

该命令会以下面的形式执行：

```bash
RVM_PATH=/home/rvm/ gem install unicorn
```

也可以在 ENV 中指定多个环境变量：

```dockerfile
ENV RVM_PATH=/home/rvm RVM_ARCHFlAGS="-arch i386"
```

在 Dockerfile 其他指令中使用这些环境变量：

```dockerfile
ENV TARGET_DIR /opt/app
WORKDIR $TARGET_DIR
```

在这里我们设置了一个新的环境变量 TARGET_DIR，并在 WORKDIR 中使用了它的值。因此实际上 WORKDIR 指令的值会被设为 /opt/app。

这些环境变量也会被持久保存到从我们的镜像创建的任何容器中。

也可以使用 `docker run` 命令行的 -e 标志指定运行时的环境变量。这些变量将只会在运行时有效：

```bash
sudo docker run -ti -e "WEB_PORT=8080" ubuntu env
```

### USER

USER 指令用来指定该镜像会以什么样的用户去运行。比如：

```dockerfile
USER nginx
```

基于该镜像启动的容器会以 nginx 用户的身份来运行。我们可以指定用户名或 UID 以及组或 GID，甚至是两者的组合。

```dockerfile
USER user
USER user:group
USER uid
USER uid:gid
USER user:gid
USER uid:group
```

也可以在 `docker run` 命令中通过 -u 标志来覆盖该指令指定的值。

如果不通过 USER 指令来指定用户，默认为 root。


### VOLUME

VOLUME 指令用来向基于镜像创建的容器添加卷。一个卷是可以存在于一个或者多个容器内的特定的目录，这个目录可以绕过联合文件系统，并提供如下共享数据或者对数据进行持久化的功能。

* 卷可以在容器间共享和通用

* 一个容器可以不是必须和其他容器共享卷

* 对卷的修改是立即生效的

* 对卷的修改不会对更新镜像产生影响

* 卷会一直存在直到没有任何容器再使用它

卷可以让我们可以将数据（如源代码）、数据库或者其他内容添加到镜像中而不是将这些内容提交到镜像中，并且允许我们在多个容器间共享这些内容。我们可以利用此功能来测试容器和内部的应用程序的代码，管理日志，或者处理容器内部的数据库。

```dockerfile
VOLUME [ "/opt/project" ]
```

这条指令将会基于此镜像创建的任何容器创建一个名为 /opt/project 的挂载点。

`docker cp` 是和 VOLUME 指令相关而且也是很实用的命令。改命令允许从容器复制文件和复制文件到容器上。

也可以通过数组的方式指定多个卷：

```dockerfile
VOLUME [ "/opt/project", "/data" ]
```


### ADD

ADD 指令用来将构建环境下的文件和目录复制到镜像中。比如，在安装一个应用程序时，ADD 指令需要源文件位置和目的文件位置两个参数：

```dockerfile
ADD software.lic /opt/application/software.lic
```

这里的 ADD 指令将会将构建目录下的 software.lic 文件复制到镜像中的 /opt/application/software.lic。指向源文件的位置参数可以是一个 URL，或者构建上下文或环境中文件名或者目录。不能对构建目录或者上下文之外的文件进行 ADD 操作。

在 ADD 文件时，Docker 通过目的地址参数末尾的字符串来判断文件源是目录还是文件。如果目标地址以 / 结尾，那么 Docker 就认为源位置指向的是一个目录。如果目的地址不是以 / 结尾，那么 Docker 就认为源位置指向的是文件。

文件源也可以使用 URL 的格式。

在 ADD 指令中使用 URL 作为文件源：

```dockerfile
ADD http://wordpress.org/latest.zip /root/wordpress.zip
```

最后值得一提的是，ADD 在处理本地归档文件（tar archive）时还有一些小魔法。如果将一个归档文件指定为源文件，Docker 会自动解开这些归档文件。

将归档文件作为 ADD 指令中的源文件：

```dockerfile
ADD latest.tar.gzz /var/www/wordpress/
```

这条命令会将归档文件 latest.tar.gz 解压到 /var/www/wordpress/ 目录下。如果目的位置的目录下已经存在了和归档文件同名的文件或者目录，那么目的位置中的文件或者目录不会被覆盖。

最后，如果目的位置不存在的话，Docker 将会为我们创建这个全路径，包括路径中的任何目录。新创建的文件和目录的模式为 0755，并且 UID 和 GID 都是 0。

> ADD 指令会使得构建缓存变得无效，这一点也非常重要。如果通过 ADD 指令向镜像添加一个文件或者目录，那么这将使 Dockerfile 中的后续指令都不能继续使用之前的构建缓存。


### COPY

COPY 指令非常类似于 ADD，它们根本的不同是 COPY 只关心在构建上下文中复制本地文件，而不会去做文件提取（extraction）和解压（decompression）的工作。

```dockerfile
COPY conf.d/ /etc/apache2/
```

这条指令会把本地 conf.d 目录中的文件复制到 /etc/apache2/ 目录中。

文件源路径必须是一个与当前构建环境相对的文件或者目录，本地文件都放到和 Dockerfile 同一个目录下。不能复制该目录之外的任何文件，因为构建环境将会上传到 Docker 守护进程，而复制是在 Docker 守护进程中进行的。任何位于构建环境之外的东西都是不可用的。COPY 指令的目的位置则必须是容器内的一个绝对路径。

任何由 COPY 指令创建的文件或者目录的 UID 和 GID 都会设置为 0。

如果源路径是一个目录，那么这个目录将整个被复制到容器中，包括文件系统元数据；如果源文件为任何类型的文件，则该文件会随同元数据一起被复制。

如果目的位置不存在，Docker 将会自动创建所有需要的目录结构，就像 mkdir -p 命令那样。


### LABEL

LABEL 指令用于为 Docker 镜像添加元数据。元数据以键值对的形式展现。

```dockerfile
LABEL version="1.0"
LABEL location="New York" type="Data Center" role="Web Server"
```

LABEL 指令以 label="value" 的形式出现。可以在每一条 LABEL 指令中指定一个元数据，或者指定多个元数据，不同的元数据之间用空格分隔。推荐将所有的元数据都放到一条 LABEL 指令中，以防止不同的元数据指令创建过多的镜像层。可以通过 `docker inspect` 命令来查看 Docker 镜像中的标签信息。

```bash
sudo docker inspect jamtur01/apache2
```


### STOPSIGNAL

STOPSIGNAL 指令用来设置停止容器时发送什么系统信号给容器。这个信号必须是内核系统调用表中合法的数，如 9，或者 SIGNAME 格式中的信号名称，如 SIGKILL。


### ARG

ARG 指令用来定义可以在 `docker build` 命令运行时传递给构建运行时的变量，我们只需要在构建时使用 `--build-arg` 标志即可。用户只能在构建时指定在 Dockerfile 文件中定义过的参数。

```dockerfile
ARG build
ARG webapp_user=user
```

上面第二条 ARG 指令设置了一个默认值，如果构建时没有为该参数指定值，就会使用这个默认值。

```bash
docker build --buil-arg build=1234 -t jamtur01/webapp .
```

这里构建 jamtur01/webapp 镜像时，build 变量将会被设置为 1234，而 webapp_user 变量则会继承设置的默认值 user。

Docker 预定义了一组 ARG 变量，可以在构建时直接使用，而不必再到 Dockerfile 中自行定义。

预定义 ARG 变量

```
HTTP_PROXY
http_proxy
HTTPS_PROXY
https_proxy
FTP_PROXY
ftp_proxy
NO_PROXY
no_proxy
```

要想使用这些预定义的变量，只需要给 `docker build` 命令传递 --build-arg <variable>=<value> 标志就可以了。


### ONBUILD

ONBUILD 指令能为镜像添加触发器（trigger）。当一个镜像被用作其他镜像的基础镜像时，该镜像中的触发器将会被执行。

触发器会在构建过程中插入新指令，我们可以认为这些指令是紧跟在 FROM 之后指定的。触发器可以是任何构建命令：

```dockerfile
ONBUILD ADD . /app/src
ONBUILD RUN cd /app/src && make
```

上面的代码将会在创建的镜像中加入 ONBUILD 触发器，ONBUILD 指令可以在镜像上运行 `docker inspect` 命令来查看。

```bash
sudo docker inspect aiqlppc6712a
```

ONBUILD 触发器会按照在父镜像中指定的顺序执行，并且只能被继承一次（也就是说只能在子镜像中执行，而不会在孙子镜像中执行）。

> 有好几条指令是不能用在 ONBUILD 指令中，包括 FROM、MAINTAINER 和 ONBUILD 本身。之所以这么规定是为了防止在 Dockerfile 构建过程中产生递归调用的问题。

