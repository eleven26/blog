---
title: Golang modules 使用
date: 2020-02-13 21:11:30
tags: [Go]
---


Golang 1.11 推出了 modules 机制来进行依赖管理。


# modules 简单使用方式

在 1.12 版本之前，使用 Go modules 之前需要环境变量 GO111MODULE:

* GO111MODULE=off: 不使用 modules 功能

* GO111MODULE=on: 使用 modules 功能，不会去 GOPATH 下面查找依赖包

* GO111MODULE=auto: Golang 自己检测是不是使用 modules 功能

在 GOPATH 之外创建一个项目 mod-demo，包含一个 main.go，内容如下：

```
package main

import (
    "github.com/astaxie/beego"
)

func main() {
    beego.Run()
}
```

## 初始化

初始化很简单，在项目根目录执行命令 `go mod init mod-demo`，然后会生成一个 go.mod 文件如下。

```
➜ mod-demo $ go mod init mod-demo
go: creating new go.mod: module .
➜ mod-demo $ ls
go.mod             main.go
➜ mod-demo $ cat go.mod
module .

go 1.12
```

这里比较关键的就是这个 go.mod 文件，这个文件标识了我们的项目的依赖的 package 的版本。执行 init 暂时
还没有将所有的依赖管理起来。我们需要将程序 run 起来（比如执行 go run/test），或者 build（执行命令
go build）的时候，才会触发依赖的解析。

比如使用 go run 即可触发 modules 工作。

```
➜ mod-demo $ go run main.go
go: extracting github.com/astaxie/beego v1.12.0
2019/09/08 23:23:03.507 [I] http server Running on http://:8080
```

这个时候我们再查看 go.mod 文件:

```
➜ mod-demo $ catgo.mod
module mod-demo

go 1.12

require (
github.com/astaxie/beego v1.12.0
github.com/shiena/ansicolor v0.0.0-20151119151921-a422bbe96644 // indirect
)
```

同时我们发现项目目录下多了一个 go.sum 用来记录每个 package 的版本和哈希值。go.mod 文件正常情况会
包含 module 和 require 模块，初次之外还可以包含 replace 和 exclude 模块。

这些 package 并不是直接存储到 $GOPATH/src，而是存储到 $GOPATH/pkg/mod 下面，不同版本并存的方式。

```
➜ mod-demo $ ls $GOPATH/pkg/mod/github.com/astaxie
beego@v1.11.0 beego@v1.11.1 beego@v1.12.0
```


## 依赖升级（降级）

可以使用如下命令来查看当前项目依赖的所有的包。

```
go list -m-uall
```

如果我想升级（降级）某个 package 则只需要 go get 即可，比如：

```
go get package@version
```

需要注意的是，在 modules 模式开启和关闭的情况下，go get 的使用方式不是完全相同的。在 modules 模式
开启的情况下，可以通过在 package 后面添加 @version 来表名要升级（降级）到某个版本。如果没有指明 version
的情况下，则默认先下载打了 tag 的 release 版本，比如 v0.4.5 或者 v1.2.3；如果没有 release 版本，则
下载最新的 pre release 版本，比如 v0.0.1-pre1。如果还没有则下载最新的 commit。这个地方给我们的
一个启示是如果我们不按规范来命名我们的 package 的 tag，则 modules 是无法管理的。version 的格式
为 v(major).(minor).(patch)，更多信息可以参考: https://semver.org/。

比如我们现在想将我们依赖中的 beego 项目的版本改为 v1.11.1，则可以像如下操作。我们发现执行完 go get 
之后，go.mod 中的项目的版本也相应的改变了。

```
➜ mod-demo $ go get github.com/astaxie/beego@v1.11.1
go: finding github.com/astaxie/beego v1.11.1
go: downloading github.com/astaxie/beego v1.11.1
go: extracting github.com/astaxie/beego v1.11.1
➜ mod-demo $ cat go.mod
module .

go 1.12

require (
github.com/OwnLocal/goes v1.0.0 // indirect
github.com/astaxie/beego v1.11.1 // indirect
github.com/shiena/ansicolor v0.0.0-20151119151921-a422bbe96644 // indirect
)
```

在 modules 开启的模式下，go get 还支持 version 模糊查询，比如 >v1.0.0 表示大于 v1.0.0 的可用版本；
<v1.12.0 表示小于 v1.12.0 版本下最近可用的版本。version 的比较规则按照 version 的各个字段来展开。

除了指定版本，我们还可以使用如下命名使用最近的可行的版本:

* go get -u 使用最新的 minor 或者 patch 版本

* go get -u=patch 使用最新的 patch 版本


## vendor

我们知道 Go 1.5 推出了 vendor 机制，go mod 也可以支持 vendor 机制，将依赖包拷贝到 vendor 目录。
但是像一些 test case 里面的依赖包并不会拷贝到 vendor 目录中。

```
go mod vendor
```


## modules 特性


### GoProxy

有些 Golang 的 package 在国内是无法直接 go get 的。在之前，我们解决这个问题，一般是通过设置 http_proxy/https_proxy 来解决。GoProxy 相当于官方提供了一种 proxy 的方式让用户来进行包下载。
要使用 GoProxy 只需要设置环境变量 GOPROXY 即可。目前公开的 GOPROXY 有：

* goproxy.io

* goproxy.cn

值得注意的是，在最新 release 的 Go 1.13 版本中默认将 GOPROXY 设置为 https://proxy.golang.org，这个对于国内的开发者是无法直接使用的。所以如果升级了 Go 1.13 版本一定要把 GOPROXY 手动改掉。


### Replace

replace 主要是为了解决某些包发生改名的问题。

对于另外一种场景有时候也是有用的，比如对于有些 golang.org/x/ 下面的包由于某些历史原因在国内是下载不了的，但是对应的包在 github 上面是有一份拷贝的，这个时候我们就可以将 go.mod 中的包进行 replace 操作。

下面是一个 Beego 项目的 go.mod 的 replace 的示例：

```
replace golang.org/x/crypto v0.0.0-20181127143415-eb0de9b17e85 => github.com/golang/crypto v0.0.0-20181127143415-eb0de9b17e85

replace gopkg.in/yaml.v2 v2.2.1 => github.com/go-yaml/yaml v0.0.0-20180328195020-5420a8b6744d

```


### SubCommand

```
go help mod
```

* download: 下载 modules 到本地缓存

* edit: 提供一种命令行交互修改 go.mod 的方式

* graph: 将 module 的依赖图在命令行打印出来，其实并不是很直观

* init: 初始化 modules，会生成一个 go.mod 文件

* tidy: 清理 go.mod 中的依赖，会添加缺失的依赖，同时移除没有用到的依赖

* vendor: 将依赖打包拷贝到项目的 vendor 目录下，值得注意的是并不会将 test code 中的依赖打包到 vendor 中。

* verify: verify 用来检测依赖包自下载之后是否被改动过。

* why: 解释为什么 package 或者 module 是需要的，但是看上去解释的理由并不是很直观。
