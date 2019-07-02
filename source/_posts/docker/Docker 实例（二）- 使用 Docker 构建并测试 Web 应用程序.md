---
title: Docker 实例（二）- 使用 Docker 构建并测试 Web 应用程序
date: 2019-06-30 16:36:00
tags: Docker
---

在本文将测试一个基于 Sinatra 的 Web 应用程序，而不是静态网站，然后我们将基于 Docker 来对这个应用进行测试。Sinatra 是一个基于 Ruby 的 Web 应用框架，它包含一个 Web 应用库，以及简单的领域专用语言（即 DSL）来构建 Web 应用程序。与其他复杂的 Web 应用框架（如 Ruby on Rails）不同，Sinatra 并不遵循 MVC 模式，而关注于让开发者创建快速、简单的 Web 应用。

因此 Sinatra 非常适合用来创建一个小型的示例应用进行测试。在这个例子里，我们将创建一个应用程序，它接收输入的 URL 参数，并以 JSON 散列的结构输出到客户端。


### 构建 Sinatra 应用程序

```bash
mkdir -p sinatra
cd sinatra
```

Dockerfile

```dockerfile
FROM ubuntu:16.04
MAINTAINER James Turnbull "james@example.com"
ENV REFRESHED_AT 2014-06-01
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update -yqq && apt-get -yqq install ruby2.3 ruby2.3-dev build-essential redis-tools
RUN gem install --no-rdoc --no-ri sinatra json redis

RUN mkdir -p /opt/webapp

EXPOSE 4567

CMD [ "/opt/webapp/bin/webapp" ]
```

可以看到，我们已经创建了另一个基于 Ubuntu 的镜像，安装了 Ruby 和 RubyGem，并且使用 gem 命令安装了 sinatra、json 和 redis gem。sinatra 是 Sinatra 的库，json 用来提供对 JSON 的支持。redis gem 会在后面会用到，用来和 Redis 数据库进行集成。

我们已经创建了一个目录用来存放新的 Web 应用程序，并公开了 WEBrick 的默认端口 4567。

最后，使用 CMD 指定 /opt/webapp/bin/webapp 作为 Web 应用程序的启动文件。

现在，使用 `docker build` 命令来创建新的镜像：

```bash
sudo docker build -t jamtur01/sinatra .
```


### 创建 Sinatra 容器

下载 Sinatra Web 应用程序：

```bash
cd sinatra
wget --cut-dirs=3 -nH -r --reject Dockerfile,index.html --no-parent http://dockerbook.com/code/5/sinatra/webapp/
ls -l webapp
```

sinatra/webapp/lib/app.rb

```rb
require "rubygem"
require "sinatra"
require "json"

class App < Sinatra::Application
    set :bind, '0.0.0.0'
    
    get '/' do
        "<h1>Dockerbook Test Sinatra app</h1>"
    end
    
    post '/json/?' do
        params.to_json
    end
end
```

可以看到，这个程序很简单，所有访问 json 端点的 POST 请求参数都会被转换为 JSON 的格式后输出。

这里还要使用 `chmod` 命令确保 webapp/bin/webapp 这个文件可执行：

```bash
chmod +x webapp/bin/webapp
```

现在我们就可以基于我们的镜像，通过 `docker run` 命令启动一个新容器。要启动容器，我们需要在 sinatra 目录下，因为我们需要将这个目录下的源代码通过卷挂载到 Dockerfile 里创建的目录 /opt/webapp。

启动第一个 Sinatra 容器：

```bash
sudo docker run -d -p 4567 --name webapp \
-v $PWD/webapp:/opt/webapp jamtur01/sinatra
```

这里从 jamtur01/sinatra 镜像创建了一个新的名为 webapp 的容器。指定了一个新卷，使用存放新 Sinatra Web 应用程序的 webapp 目录，并将这个卷挂载到在 Dockerfile 里创建的指定目录 /opt/webapp。

我们没有指定要运行的命令，而是使用在镜像的 Dockerfile 中 CMD 指令设置的命令：

```dockerfile
CMD [ "/opt/webapp/bin/webapp" ]
```

从这个镜像启动容器时，将会执行这一命令。

也可以使用 `docker logs` 命令查看被执行的命令都输出了什么：

```bash
sudo docker logs webapp
```

运行 `docker logs` 命令时加上 -f 标志可以达到与执行 `tail -f` 命令一样的效果 -- 持续输出到容器的 STDERR 和 STDOUT 里的内容：

```bash
sudo docker logs -l webapp
```

可以使用 `docker top` 命令查看 Docker 容器里正在运行的进程：

```bash
sudo docker top webapp
```

从这一日志可以看出，容器中已经启动了 Sinatra，而且 WEBrick 服务进程正在监听 4567 端口，等待测试。先查看一下这个端口映射到本地宿主机的哪个端口：

```bash
sudo docker port webapp 4567
```

测试容器：

```bash
curl -i -H 'Accept: application/json' \
-d 'name=Foo&status=Bar' http://localhost:49160/json
```


### 扩展 Sinatra 应用程序来使用 Redis

现在我们将要扩展 Sinatra 应用程序，加入 Redis 后端数据库，并在 Redis 数据库中存储输入的 URL 参数。

##### 升级 Sinatra 应用程序

```bash
cd sinatra
wget --cut-dirs=3 -nH -r --reject Dockerfile,index.html --no-parent http://dockerbook.com/code/5/sinatra/webapp_redis/
```

app.rb

```ruby
require "rubygems"
require "sinatra"
require "json"
require "redis"

class App < Sinatra::Application
    redis = Redis.new(:host => 'db', :port => '6379')
    
    set :bind, '0.0.0.0'
    
    get '/' do
        "<h1>DockerBook Test Redis-enabled Sinatra app</h1>"
    end
    
    get '/json' do
        params = redis.get "params"
        params.to_json
    end
    
    post '/json/?' do
        redis.set "params", [params].to_json
        params.to_json
    end
end
```

新版本的代码只是增加了对 Redis 的支持。我们创建了一个到 Redis 数据库的连接，用来连接名为 db 的宿主机上的 Redis 数据库，端口为 6379。我们在 POST 请求处理中，将 URL 参数保存到了 Redis 数据库中，并在需要的时候通过 GET 请求取回这个值。


### 构建 Redis 数据库镜像

为了构建 Redis 数据库，要创建一个新的镜像。

```bash
mkdir -p sinatra/redis
cd sinatra/redis
```

用于 Redis 镜像的的 Dockerfile

```dockerfile
FROM ubuntu:14.04
MAINTAINER James Turnbull "james@example.com"
ENV REFRESHED_AT 2014-06-01
RUN apt-get -yqq update && apt-get -yqq install redis-server redis-tools
EXPOSE 6379
ENTRYPOINT [ "/usr/bin/redis-server" ]
CMD []
```

我们在 Dockerfile 里指定了安装 Redis 服务器，公开 6379 端口，并指定了启动 Redis 服务器的 ENTRYPOINT。现在来构建这个镜像：

```bash
sudo docker build -t jamtur01/redis .
```

启动 Redis 容器：

```bash
sudo docker run -d -p 6379 --name redis jamtur01/redis
```

可以看到，我们从 jamtur01/redis 镜像启动了一个新的容器，名字是 redis。注意，我们指定了 -p 标志来公开 6379 端口。看看这个端口映射到宿主机的哪个端口：

```bash
sudo docker port redis 6379
```


### 将 Sinatra 应用程序连接到 Redis 容器

现在来更新 Sinatra 应用程序，让其连接到 Redis 并存储传入的参数。为此，需要能够与 Redis 服务器对话。要做到这一点，可以用以下几种方法：

* Docker 的内部网络

* 从 Docker1.9 及之后的版本开始，可以使用 Docker Networking 以及 docker network 命令

* Docker 链接。一个可以将具体容器链接到一起来进行通信的抽象层。

如果用户使用 Docker1.9 或者更新的版本，推荐使用 Docker Networking。

在 Docker Networking 和 Docker 链接之间也有一些区别：

* Docker Networking 可以将容器连接到不同宿主机上的容器。

* 通过 Docker Networking 连接的容器可以在无须更新连接的情况下，对停止、启动或者重启容器。而使用 Docker 链接，则可能需要更新一些配置，或者重启相应的容器来维护 Docker 容器之间的链接。

* 使用 Docker Networking，不必事先创建容器再去连接它。同样，也不必关心容器的运行顺序，读者可以在网络内部获得容器名解析和发现。


### Docker 内部连网

第一种方法涉及 Docker 自己的网络栈。到目前为止，我们看到的 Docker 容器都是公开端口并绑定到本地网络接口的，这样可以把容器里的服务在本地 Docker 宿主机所在的外部网络上公开。除了这种用法，Docker 这个特性还有种用法我们没有见过，那就是内部网络。

在安装 Docker 时，会创建一个新的网络接口，名字是 docker0。每个 Docker 容器都会在这个接口上分配一个 IP 地址。

```bash
ip a show docker0
```

可以看到，docker0 接口有符合 RFC1918 的私有 IP 地址，范围是 172.16~172.30。接口本身的地址是 172.17.42.1 是这个 Docker 网络的网关地址，也是所有 Docker 容器的网关地址。

> Docker 会默认使用 172.17.x.x 作为子网地址，除非已经有别人占用了这个子网。如果这个子网被占用了，Docker 会在 172.16~172.30 这个范围内尝试创建子网。

接口 docker0 是一个虚拟的以太网桥，用于连接容器和本地宿主网络。如果进一步查看 Docker 宿主机的其他网络接口，会发现一系列以 veth 开头的接口。

Docker 每创建一个容器就会创建一组互联的网络接口。这组接口就像管道的两端（就是说，从一端发送的数据会在另一端接收到）。这组接口其中一端作为容器里的 eth0 接口，而另一端统一命名为类似 vethec6a 这组名字，作为宿主机的一个端口。可以把 veth 接口认为是虚拟网线的一端。这个虚拟网线一端插在名为 docker0 的网桥上，另一端插到容器里。通过把每个 veth* 接口绑定到 docker0 网桥，Docker 创建了一个虚拟子网，这个子网由宿主机和所有的 Docker 容器共享。

进入容器里面，看看这个子网管道的另一端：

```bash
sudo docker run -t -i ubuntu /bin/bash
```

可以看到，Docker 给容器分配了 IP 172.17.0.29 作为宿主虚拟接口的另一端。这样就能够让宿主网络和容器相互通信了。

让我们从容器内跟踪对外通信的路由，看看是如何建立连接的：

```bash
apt-get -yqq update && apt-get install -yqq traceroute
traceroute google.com
```

可以看到，容器地址的下一跳是宿主网络上 docker0 接口的网关 IP 172.17.42.1

不过 Docker 网络还有还有另外一个部分配置才允许建立连接：防火墙规则和 NAT 配置。这些配置允许 Docker 在宿主网络和容器间路由。现在来查看一下宿主机上的 IPTables NAT 配置：

```bash
sudo iptables -t nat -L -n
```

这里还有几个值得注意的 IPTables 规则。首先，我们注意到，容器默认是无法访问的。从宿主网络与容器通信时，必须明确指定打开的端口。


### Redis 容器的网络

使用 `docker inspect` 命令查看新的 Redis 容器的网络配置：

```bash
sudo docker inspect redis
```

`docker inspect` 命令展示了 Docker 容器的细节，这些细节包括配置信息和网络状况。也可以在命令里使用 -f 标志只获取 IP 地址：

```bash
sudo docker inspect -f '{{ .NetworkSettings.IPAddress }}' redis
```

通过运行 `docker inspect` 命令可以看到容器的 IP 地址。还能看到宿主机和容器之间的端口映射关系。同时，因为运行在本地的 Docker 宿主机上，所以不一定要用映射后的端口，也可以直接使用分配给 redis 容器的 IP 和容器的 6379 端口。

```bash
redis-cli -h 172.17.0.18
```

Docker 默认会把公开的端口绑定到所有的网络接口上。因此，也可以通过 localhost  或者 127.0.0.1 来访问 Redis 服务器。

这种容器互联的方案存在的问题：

* 要在应用程序里对 Redis 容器的 IP 地址做硬编码

* 如果重启容器，Docker 会改变容器的 IP 地址（比如 `docker restart` 之后，容器 IP 地址会发生改变）


### Docker Networking

容器之间的连接用网络创建，这被称为 Docker Networking。Docker Networking 允许用户创建自己的网络，容器可以通过这个网络互相通信。实质上，Docker Networking 以新的用户管理的网络补充了现有的 docker0。更重要的是，现在容器可以跨越不同的宿主机来通信，并且网络配置可以更灵活地定制。Docker Networking 也和 Docker Compose 以及 Swarm 进行了集成。

> Docker Networking 支持也是可以插拔的，也就是说可以增加网络驱动以支持来自不同网络设备提供商的特定拓扑和网络框架

创建一个 Docker 网络：

```bash
sudo docker network create app
```

这里用 `docker network` 命令创建了一个桥接网络，命名为 app，这个命令返回新创建的网络的网络 ID。

然后可以用 `docker network inspect` 命令查看新创建的这个网络：

```bash
sudo docker network inspect app
```

我们可以看到这个新网络是一个本地的桥接网络。

> 除了运行于单个主机上的桥接网络，我们也可以创建一个 overlay 网络，overlay 网络允许我们跨多台宿主机进行通信。

可以使用 `docker network ls` 列出当前系统中的所有网络：

```bash
sudo docker network ls
```

也可以使用 `docker network rm` 删除一个 Docker 网络。


在 Docker 网络中创建 Redis 容器：

```bash
sudo docker rum -d --net=app --name db jamtur01/redis
```

这里我们基于 jamtur01/redis 镜像创建了一个名为 db 的新容器。我们同时指定了一个新的标志 --net，--net 标志指定了新容器将会在哪个网络中运行。

这时，如果再次运行 `docker network inspect` 命令，将会看到这个网络更详细的信息：

```bash
sudo docker network inspect app
```

链接 Redis 容器：

```bash
cd sinatra/webapp
sudo docker run -p 4567 \
--net=app --name webapp -t -i \
-v $PWD/webapp:/opt/webapp jamtur01/sinatra \
/bin/bash
```

我们在 app 网络下启动了一个名为 webapp 的容器。

由于这个容器是在 app 网络内部启动的，因此 Docker 将会感知到所有在这个网络下运行的容器，并且通过 /etc/hosts 文件，将这些容器的地址保存到本地 DNS 中。

```bash
cat /etc/hosts
```

我们可以看到有一条记录是 `172.18.0.2 db`，这就意味着我们可以通过 db 来访问 redis 容器。

代码中指定 Redis DB 主机名：

```ruby
redis = Redis.new(:host => 'db', :port => '6379')
```

现在，就可以启动我们的应用程序，并且让 Sinatra 应用程序通过 db 和 webapp 两个容器间的连接，将接收到的参数写入 Redis 中，db 和 webapp 容器间的连接也是通过 app 网络建立的。

重要的是，如果任何一个容器重启了，那么它们的 IP 地址信息也会在网络其他容器中的 /etc/hosts 更新。也就是说，对底层容器的修改并不会对我们的应用程序正常工作产生影响。


##### 将已有容器连接到 Docker 网络



