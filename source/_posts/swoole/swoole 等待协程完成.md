---
title: swoole 等待协程完成
date: 2019-12-11 09:05:00
tags: [swoole]
---

> 几番折腾，实现不了想要的效果（在 swoole worker 里面使用多个协程发送 MySQL 查询，不同协程使用不同的 MySQL 连接，应用里面维护一个连接池，然后等待这些协程结束），下面的内容可能有部分不正确。

> 实际使用过程中，要么异步了，但是没有等待直接往下执行。要么等待了，但是也直接阻塞了。

> 因为 Reactor 线程不在 worker 进程


## 2019-12-24 更新

实现方法还是有的，使用 swoole 的协程 Http server、协程 MySQL 客户端


## swoole_event_wait

使用场景：单个进程。同时发出多个 http 请求，然后等待所有 http 请求返回再往下执行。

```
// 在单进程情况下，使用 swoole_event_wait
$a = 1;

go(function() use (&$a) {
    co::sleep(1); // 模拟 io 阻塞
    $a++;
    echo $a . PHP_EOL;
});

$a = $a + 3;
echo $a . PHP_EOL;
swoole_event_wait();

$a = $a + 10;
echo $a . PHP_EOL;
```

输出:

```
4
5
15
```

!!! 在这种情形下使用 `swoole_event_dispatch` 是不会有阻塞效果的。使用 `swoole_event_dispatch` 的输出:

```
4
14
15
```

## swoole_event_dispatch

使用场景：在 swoole server 的 worker 进程里面需要等待协程。比如和现在的 [laravel-s](https://github.com/hhxsv5/laravel-s) 框架集成的时候

```
// 工作在 swoole worker 下，使用 swoole_event_dispatch
$a = 1;

go(function() use (&$a) {
    co::sleep(1); // 模拟 io 阻塞
    $a++;
    echo $a . PHP_EOL;
});

$a = $a + 3;
echo $a . PHP_EOL;
swoole_event_dispatch();

$a = $a + 10;
echo $a . PHP_EOL;
```

输出:

```
4
5
15
```

## swoole_event_wait 与 swoole_event_dispatch 的区别

相关文档：

[swoole_event_dispatch](https://wiki.swoole.com/wiki/page/p-swoole_event_dispatch.html)

`swoole_event_wait` 与 `swoole_event_dispatch` 不同的是，`swoole_event_wait` 在底层内部维持了循环。

`swoole_event_dispatch` 在框架内部执行控制 reactor 的循环，而使用 `swoole_event_wait`，
swoole 底层底层维持了控制权，就无法让出给框架方。

由于这个原因，在 swoole server 的 worker 进程里面使用 `swoole_event_wait` 有可能会导致进程直接阻塞。


## ~~等待多个协程~~

这里有误，暂时不知道原因

~~如果我们有多个协程需要等待，则我们需要在协程后面多次调用 `swoole_event_dispatch` 或者 `swoole_event_wait`。~~

~~一个协程对应一个 `swoole_event_dispatch` 或者 `swoole_event_wait`。~~

如：

```
$a = 1;

go(function() use (&$a) {
    co::sleep(0.04); // 模拟 io 阻塞
    $a ++;
    echo $a . PHP_EOL;
});

$a = $a + 3;
echo $a . PHP_EOL;
swoole_event_wait();  // 等待上一个协程完成

go(function () use (&$a) {
    co::sleep(0.3);
    $a+=4;
    echo $a . PHP_EOL;
});
swoole_event_wait(); // 等待上一个协程完成

$a = $a + 10;
echo $a . PHP_EOL;
```

输出:

```
4
5
9
19
```

> 注意事项: 工作在 worker 进程下时候，不能一个协程后面写多个 `swoole_event_dispatch`，否则会导致进程阻塞。


## 没有任何事件发生会一直阻塞

如果我们在 swoole worker 的 onRequest 里面调用了 swoole_event_dispatch，但是没有任何事件发生，进程会一直阻塞。
