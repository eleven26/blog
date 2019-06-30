---
title: Docker 实例（一） - 使用 Docker 测试静态网站.md
date: 2019-06-30 13:05:02
tags: Docker
---

### Sample 网站的初始 Dockerfile

为 Nginx Dockerfile 创建一个目录

```bash
mkdir sample
cd sample
touch Dockerfile
```

获取 Nginx 配置文件

```bash
mkdir nginx && cd nginx
wget https://raw.githubusercontent.com/jamtur01/dockerbook-code/master/code/5/sample/nginx/global.conf
wget https://raw.githubusercontent.com/jamtur01/dockerbook-code/master/code/5/sample/nginx/nginx.conf
```

网站测试的基本 Dockerfile

```dockerfile
FROM ubuntu:14.04
MAINTAINER James Turnbull "james@example.com"
ENV REFRESHED_AT 2014-06-01
RUN apt-get -yqq update && apt-get -yqq install nginx
RUN mkdir -p /var/www/html/website
ADD nginx/global.conf /etc/nginx/conf.d/
ADD nginx/nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
```

这个简单的 Dockerfile 内容包括以下几项。

* 安装 nginx

* 在容器中创建一个目录 /var/www/html/website/

* 将来自我们下载的本地文件的 Nginx 配置文件添加到镜像中

* 公开镜像的 80 端口

这个 Nginx 配置文件是为了运行 Sample 网站而配置的。将文件 nginx/global.conf 用 ADD 指令复制到 /etc/nginx/conf.d/ 目录中。配置文件 global.conf 的内容如下：

```
server {
    listen 0.0.0.0:80;
    server_name _;
    
    root /var/www/html/website;
    index index.html index.htm;
    
    access_log /var/log/nginx/default_access.log;
    error_log /var/log/nginx/default_error.log;
}
```

这个文件将 Nginx 设置为监听 80 端口，并将网络服务的根路径设置为 /var/www/html/website，这个目录是我们用 RUN 指令创建的。

我们还需要将 Nginx 配置为非守护进程的模式，这样可以让 Nginx 在 Docker 容器里工作。将文件 nginx/nginx.conf 复制到 /etc/nginx 目录就可以达到这个目的，nginx.conf 的文件内容如下：

```
user www-data;
worker_processes 4;
pid /run/nginx.pid;
daemon off;

events {}

http {
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    include /etc/nginx/mime.typs;
    default_type application/octet-stream;
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    gzip on;
    gzip_disable "msie6";
    include /etc/nginx/conf.d/*.conf;
}
```

在这个配置文件里，`daemon off;` 选项阻止 Nginx 进入后台，强制其在前台运行。这是因为想要保持 Docker 容器的活跃状态，需要其中运行的进程不能中断。默认情况下，Nginx 会以守护进程的方式启动，这会导致容器只是短暂运行，在守护进程被 fork 后，发起守护进程的原始进程就会退出，这时容器就停止运行了。

这个文件通过 ADD 指令复制到 /etc/nginx/nginx.conf。


### 构建 Sample 网站和 Nginx 镜像

利用之前的 Dockerfile，可以用 `docker build` 命令构建出新的镜像，并将这个镜像命名为 jamtur01/nginx：

```bash
sudo docker build -t jamtur01/nginx .
```

这将构建并命名一个新镜像。下面来看看构建的执行步骤。使用 `docker history` 命令查看构建新镜像的步骤和层级：

```bash
sudo docker history jamtur01/nginx
```

history 命令从新构建的 jamtur01/nginx 镜像的最后一层开始，追溯到最开始的父镜像 ubuntu14.04。这个命令也展示了每步之间创建的新层，以及创建这个层所使用的 Dockerfile 里的指令。


### 从 Sample 网站和 Nginx 镜像构建容器

现在可以使用 jamtur01/nginx 镜像，并开始从这个镜像构建可以用来测试 Sample 网站的容器。为此，需要添加 Sample 网站的代码。

```bash
mkdir website && cd website
wget https://raw.githubusercontent.com/jamtur01/dockerbook-code/master/code/5/sample/website/index.html
cd ..
```

这将在 sample 目录中创建一个名为 website 的目录，然后为 Sample 网站下载 index.html 文件，放到 website 目录中。

现在来看看如何使用 `docker run` 命令来运行一个容器：

```bash
sudo docker run -d -p 80 --name website -v $PWD/website:/var/www/html/website jamtur01/nginx nginx 
```

> 可以看到，在 `docker run` 时传入了 nginx 作为容器的启动命令。一般情况下，这个命令无法让 nginx 以交互的方式运行。我们已经在提供给 Docker 的配置里加入了指令 `daemon off`，这个指令让 nginx 启动后以交互的方式在前台运行。

* 卷

卷在 Docker 里非常重要，也很有用。卷是在一个或多个容器内被选定的目录，可以绕过分层的联合文件系统，为 Docker 提供持久数据或者共享数据。这意味着对卷的修改会直接生效，并绕过镜像。当提交或者创建镜像时，卷不会被包含在镜像里。

> 卷可以在容器间共享。即使容器停止，卷里的内容依旧存在。

回到刚才的例子。当我们因为某些原因不想把应用或者代码构建到镜像中时，就体现出卷的价值了。例如：

* 希望同时对代码做开发和测试

* 代码改动很频繁，不想在开发过程中重构镜像

* 希望在多个容器间共享代码

-v 选项通过指定一个目录或登上与容器上与该目录分离的本地宿主机来工作，这两个目录用 : 分隔。如果容器目录不存在，Docker 会自动创建一个。

也可以通过在目录名后面加上 rw 或者 ro 来指定容器内目录的读写状态。

```bash
sudo docker run -d -p 80 --name website \
-v $PWD/website:/var/www/html/website:ro \
jamtur01/nginx nginx
```

这将使目的目录 /var/www/html/website 变为只读状态。

在 Nginx 网站容器里，我们通过卷将 $PWD/website 挂载到容器的 /var/www/html/website 目录，顺利挂载了正在开发的本地网站。在 Nginx 配置里，已经指定了这个目录为 Nginx 服务器的工作目录。

现在，如果使用 `docker ps` 命令查看正在运行的容器，可以看到名为 website 的容器正处于活跃状态，容器的 80 端口被映射到宿主机的一个端口上：

```bash
sudo docker ps -l
```
