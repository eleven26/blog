---
title: swoole 等待协程完成
date: 2019-12-11 09:05:00
tags: [swoole]
---

解决方法：使用 `swoole_event_dispatch()` 或者 `swoole_event_wait()`


## swoole_event_wait

使用场景：单个进程。同时发出多个 http 请求，然后等待所有 http 请求返回再往下执行。

```
// 在单进程情况下，使用 swoole_event_wait
$a = 1;

go(function() use (&$a) {
    co::sleep(1); // 模拟 io 阻塞
    echo $a . PHP_EOL;
});

$a = $a + 3;
echo $a . PHP_EOL;
swoole_event_wait();

$a = $a + 10;
echo $a . PHP_EOL;
```

## swoole_event_dispatch

使用场景：在 swoole server 的 worker 进程里面需要等待协程。

```
// 工作在 swoole worker 下，使用 swoole_event_dispatch
$a = 1;

go(function() use (&$a) {
    co::sleep(1); // 模拟 io 阻塞
    echo $a . PHP_EOL;
});

$a = $a + 3;
echo $a . PHP_EOL;
swoole_event_dispatch();

$a = $a + 10;
echo $a . PHP_EOL;
```

## swoole_event_wait 与 swoole_event_dispatch 的区别

相关文档：

[swoole_event_dispatch](https://wiki.swoole.com/wiki/page/p-swoole_event_dispatch.html)

`swoole_event_wait` 与 `swoole_event_dispatch` 不同的是，`swoole_event_wait` 在底层内部维持了循环。

`swoole_event_dispatch` 在框架内部执行控制 reactor 的循环，而使用 `swoole_event_wait`，
swoole 底层底层维持了控制权，就无法让出给框架方。

由于这个原因，在 swoole server 的 worker 进程里面使用 `swoole_event_wait` 有可能会导致进程直接阻塞。
