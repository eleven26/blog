---
title: tornado 初体验
date: 2019-12-21 23:17:00
tags: [Python, tornado]
---

最近遇到一种场景，php 环境下，一个 HTTP 请求内需要查询的次数过多，但是又不得不查。

最先想到的当然是缓存，但是东西真的太多了，做缓存维护成本是非常高的。

> 有个前提：需要查询的一部分数据不需要有先后顺序的依赖。

所以最近在寻找缓存之外的一些解决办法，先是想到了使用 swoole 的协程来同时发出多个查询，同时开启多个协程，在协程里面发出查询，
但是要这样用的话，因为 swoole 的 HTTP server 工作在 worker 进程，而协程调度在 master 进程，在这个过程中发现，其实 swoole
并不能实现自己想要的结果，因为 swoole http worker 进程里面没有办法自主控制协程调度，
所以没有办法做到说，等某几个协程完成再进行下一步操作。

但不在 http server 模式下的时候，比如单一的 cli 程序，是可以实现的，因为只有一个进程。但这并不是想要的结果。

所以只有继续寻求其他的解决方案。在此过程也看了类似 swoft 的框架，发现最多只能实现 http server 层面的协程调度，
也就是说，最多只能做到，同时接受多个请求（比如，大于 CPU 总数），其中某一个请求产生 IO 事件的时候，让出 CPU，
处理其他请求，然后 IO 事件完成的时候，再继续处理。

这样有个问题是，虽然整体并发量提升了，但是单一请求时间过长的问题还是会存在的。

后来想想，其实其他语言也是可以的，然后尝试了一下 Go，发现 go 里面的协程可以用 sync 来实现协程等待。
但是 Go 毕竟语法不是太友好，然后想想 Java，但是 Java 好像又太庞大了。

后来也想到了 Python，想起 Python 那个 Tornado，然后就尝试了一下。

说了那么多，正文开始：

## 目标

同一个 http 请求内，同时发出多个查询请求，并等待这些请求返回。

## 实现方式

数据库连接池 + 协程

开启多个协程，每个协程发出一个查询，每个查询耗时 1s，最后总耗时 1s

## 代码

```
import tornado.ioloop
import tornado.gen
import tornado.web

from tornado_mysql import pools


pools.DEBUG = True


POOL = pools.Pool(
    dict(host='192.168.2.1', port=3306, user='xx', passwd='xx', db='xx'),
    max_idle_connections=1,
    max_recycle_sec=3)


class MainHandler(tornado.web.RequestHandler):
    @tornado.gen.coroutine
    def get(self):
        # 开启 10 个协程
        workers = [self.worker(i) for i in range(10)]
        yield workers

    @tornado.gen.coroutine
    def worker(self, n):
        t = 1
        print('n = {}'.format(n))
        # 每个协程的查询耗时 1s
        res = POOL.execute("SELECT SLEEP(%s)", (t,))
        print(res)
        yield res


def make_app():
    return tornado.web.Application([
        (r"/", MainHandler),
    ])


if __name__ == "__main__":
    app = make_app()
    app.listen(8888)
    tornado.ioloop.IOLoop.current().start()
```

这里还没有做结果的处理。因为还不知道怎么做
