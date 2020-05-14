---
title: Laravel queue worker 进程退出的原因
date: 2020-05-14 13:36:00
tags: [Laravel, PHP]
---

环境：RabbitMQ + Lumen 5.5，消费者处理逻辑是在处理完消息之后 ack。

Worker 源码路径 \Illuminate\Queue\Worker。

今天突然发现一个 qa 环境的一个队列消息累积了几百万，最终发现是因为消费消息的 Worker 进程没有处理完消息就退出了处理，而且没有任何的记录。看源码发现 worker 里面有个 `kill` 函数，里面执行了 `exit()` 函数，我们都知道，这个函数是退出进程的。

```PHP
/**
 * Kill the process.
 *
 * @param  int  $status
 * @return void
 */
public function kill($status = 0)
{
    $this->events->dispatch(new Events\WorkerStopping);

    if (extension_loaded('posix')) {
        posix_kill(getmypid(), SIGKILL);
    }

    exit($status);
}
```

退出其实问题不大，但是这个退出的逻辑并没有标记这个 job 失败，这就导致下一次 Worker 进程启动的时候，继续拿到这个消息处理，然后处理到一定时候，又退出，如此无限循环。


## Worker exit 的原因

我们可以看看 Worker 里面所有 exit 的调用，其实有两个地方：

1. stopIfNecessary，kill 和 stop 函数最终都是 exit。

```PHP
/**
 * Stop the process if necessary.
 *
 * @param  \Illuminate\Queue\WorkerOptions  $options
 * @param  int  $lastRestart
 */
protected function stopIfNecessary(WorkerOptions $options, $lastRestart)
{
    if ($this->shouldQuit) {
        $this->kill();
    }

    if ($this->memoryExceeded($options->memory)) {
        $this->stop(12);
    } elseif ($this->queueShouldRestart($lastRestart)) {
        $this->stop();
    }
}
```

shouldQuit 是在一些连接断开的时候被设置为 true，又或者用户发送了 SIGTERM 信号给 Worker。

2. registerTimeoutHandler，超时处理。

```PHP
/**
 * Register the worker timeout handler (PHP 7.1+).
 *
 * @param  \Illuminate\Contracts\Queue\Job|null  $job
 * @param  \Illuminate\Queue\WorkerOptions  $options
 * @return void
 */
protected function registerTimeoutHandler($job, WorkerOptions $options)
{
    if ($this->supportsAsyncSignals()) {
        // We will register a signal handler for the alarm signal so that we can kill this
        // process if it is running too long because it has frozen. This uses the async
        // signals supported in recent versions of PHP to accomplish it conveniently.
        pcntl_signal(SIGALRM, function () {
            $this->kill(1);
        });

        pcntl_alarm(
            max($this->timeoutForJob($job, $options), 0)
        );
    }
}
```

所以总结一下有以下几个原因：

* 数据库连接或其他连接断开
* Worker 接收到 SIGTERM 信号
* Worker 处理完一个 Job 之后发现 Worker 占用的内存超出了指定的内存
* 用户执行了队列重启命令
* **Worker 执行时间超出了 timeout**


## exit 的影响

1、如果使用的是 redis

如果这个消息需要消费者处理的时间大于指定的 timeout，会导致消息没处理完就丢失。

2、如果使用的是 RabbitMQ，并且关闭了 AutoAck

Worker 进程没处理完就退出，然后消息还在队列中，下次启动 Worker 的时候继续消费这个消息，导致无限的重复消费。


## 解决方案

1. 评估一下 Job 的最大运行时间，设置一个合适的 timeout，这个是必须的。
2. 可以监听 WorkerStopping 事件，记录 Worker 异常退出的日志，但是需要注意的是，正常退出也会 fire 这个事件。所以有可能没有办法根据 log 来判断是否是异常退出（超时）。
3. 这是 5.8 以下版本的 bug，我们可以升级到 5.8 以上的版本，在新版本中超时也会记录为失败，而不是单纯地退出。


## 这是个 bug

我们通过上面的代码也可以发现，其实 Worker 的超时回调其实并没有多少实际的处理，dispatch 一个 WorkerStopping 事件然后就 exit 了。但是我们有可能并没有监听这个事件，这就导致了 Worker 存在 timeout 过短的问题难以被及时发现。

其实这个 5.8 版本以下的 bug，在 5.8 以上的版本中这个已经修复了，超时的时候，Job 会被标记为超时，超过重试次数就被记录为失败的 job。

源码可在 [https://github.com/laravel/framework/blob/5.8/src/Illuminate/Queue/Worker.php#L137](https://github.com/laravel/framework/blob/5.8/src/Illuminate/Queue/Worker.php#L137) 查看。

```PHP
/**
 * Register the worker timeout handler.
 *
 * @param  \Illuminate\Contracts\Queue\Job|null  $job
 * @param  \Illuminate\Queue\WorkerOptions  $options
 * @return void
 */
protected function registerTimeoutHandler($job, WorkerOptions $options)
{
    // We will register a signal handler for the alarm signal so that we can kill this
    // process if it is running too long because it has frozen. This uses the async
    // signals supported in recent versions of PHP to accomplish it conveniently.
    pcntl_signal(SIGALRM, function () use ($job, $options) {
        if ($job) {
            $this->markJobAsFailedIfWillExceedMaxAttempts(
                $job->getConnectionName(), $job, (int) $options->maxTries, $this->maxAttemptsExceededException($job)
            );
        }

        $this->kill(1);
    });

    pcntl_alarm(
        max($this->timeoutForJob($job, $options), 0)
    );
}
```


## 其他问题

1. Job 里面写死了 timeout 属性，会以这个 timeout 为准。

在调试过程中，发现明明 `php artisan queue:work --timeout=` 这里设置的 timeout 足够大了，但是 Worker 还是和原来一样退出了。在 RabbitMQ 的控制台发现消息里面有记录 Job 的 timeout 属性，然后 Worker 里面在判断到如果 job 里面有 timeout 属性的时候，就不会再使用命令行传递的 timeout。

```PHP
/**
 * Get the appropriate timeout for the given job.
 *
 * @param  \Illuminate\Contracts\Queue\Job|null  $job
 * @param  \Illuminate\Queue\WorkerOptions  $options
 * @return int
 */
protected function timeoutForJob($job, WorkerOptions $options)
{
    return $job && ! is_null($job->timeout()) ? $job->timeout() : $options->timeout;
}
```
