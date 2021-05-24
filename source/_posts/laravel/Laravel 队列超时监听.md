---
title: Laravel 队列超时监听
date: 2021-05-24 11:04:00
tags: [Laravel, PHP]
---

在 Laravel 5.8 以下的版本中，队列消费者处理消息超时的时候会静默失败，即使超过了重试次数，这就导致一个问题，超时的时候问题无法被及时发现。

在 5.8 及以上版本中，超时次数重试次数之后也会当作失败处理，会记录到 failed_jobs 表里面，而不像之前版本那样一直重试。

具体可以查看 `\Illuminate\Queue\Worker::registerTimeoutHandler` 方法。

不过在 5.8 以下的版本中，我们依然有办法可以得知队列超时，`registerTimeoutHandler` 里面在超时退出消费者进程之前，先 fire 了一个 WorkerStopping 事件，并且传递了参数 1。

也就是说，我们可以通过监听 WorkerStopping 事件，然后通过判断 WorkerStopping 的 status 状态字段是否为 1，如果是 1，则表明队列超时了，这个时候我们就可以采取一些行动，比如发送报警通知，及时告知开发者。
