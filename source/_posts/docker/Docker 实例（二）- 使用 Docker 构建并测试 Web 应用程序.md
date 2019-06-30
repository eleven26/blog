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



