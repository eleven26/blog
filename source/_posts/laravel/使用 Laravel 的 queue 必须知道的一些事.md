---
title: 使用 Laravel 的 queue 必须知道的一些事
date: 2018-07-26 07:53：00
tags: [Laravel, PHP]
---

1. 在修改 `queue` 相关代码后，必须要使用 `php artisan queue:restart` 来重启队列服务，否则所做的修改可能不会生效(没法重现了，按理说应该和使用 `queue:listen` 或 `queue:work` 有关，不过最好还是重启；可能和 `supervisor` 开启多个 `queue:work` 进程也有关系，本地测试的时候只有一个进程)。

文档：
![x](/images/queue/1.png)


2、开发环境下以同步的方式执行队列，将 `queue driver` 的值改为 `sync`，注意，如果 `queue` 有输出的话，可能会导致一些问题，如：本来应该只返回 `json` 串的，然后 `queue` 里面有输出，导致前端 `json` 解析失败。

 
3、什么时候使用 `queue:listen` ？什么时候使用 `queue:work` ？

官网文档有一段描述是：

在 `queue:work` Artisan 命令里包含了 `--daemon` 选项，强制队列服务器持续处理任务，而不需要重新启动整个框架。比起 `queue:listen` 命令，这将明显的减少 CPU 的用量。

使用 `queue:work` 的时候不需要重现启动整个框架，这可能是 1 中可能修改 Job 后不生效的问题。

 

4、多个项目同时部署时候的冲突

Laravel 中队列任务使用 redis 驱动情况下保存的时候的缓存 key 是不带前缀的，比如 A 项目 `dispatch` 了一个 a job，保存在了 `queues:default`，然后我们去 B 项目 `dispatch` 另一个 job，我们发现它们保存在了相同的 `redis key` 中

![x](/images/queue/2.png)
![x](/images/queue/3.png)

这样会导致的问题是：在一个项目中跑 `php artisan queue:work` 会拿到另外一个项目的 job，这样就会导致一些不必要的异常，因为在反序列化的过程中会找不到对应的类。 

原因：`config/queue.php` 中配置的默认 `queue` 都是 `default`：

```php
'redis' => [
    'driver' => 'redis',
    'connection' => 'default',
    'queue' => 'default',
    'expire' => 60,
]
```

解决办法：

a、自己用的是 5.1 版本，网上有说可以修改 `cache prefix` 解决，但是 `laravel 5.1` 行不通，可能新版本可以

b、为 job 指定不同的 queue，如 `dispatch job` 的时候可以 `dispatch((new xxJob())->onQueue('xxQueue'))`，这样一来，job 就保存在了 `queues:xxQueue` 中，
但是还是得注意，如果还有其他项目，不要取相同名字。同时，这样一来，我们的 `queue:work` 或者 `queue:listen` 命令也要加上 `--queue` 参数了，
如 `php artisan queue:work --queue=testQueue`，否则还是会去 `queues:default` 里面找。如下：

![x](/images/queue/4.png)

c、直接修改 `config/queue.php`，修改 `redis.queue` 为一个唯一的名字。如：

```php
'redis' => [
    'driver' => 'redis',
    'connection' => 'default',
    'queue' => 'peper',
    'expire' => 60,
]
```

![x](/images/queue/5.png)

这样一来，我们的 job 就不会和其他项目的混在一起了。

个人看法：最好的实践应该还是，不同的队列使用不同的名字（即使是同一个项目），这样会更便于管理。


5. 给队列的 Job 对象设置模型对象属性的时候，最后处理队列的时候会重新查询这个模型的数据。

详细见：`\Illuminate\Queue\SerializesModels`

也就是说，我们如果想在新建 Job 实例的时候，通过 `setAttribute` 设置了一个模型实例的属性，想在 `handle` 里面获取这个属性的话，是获取不到的。因为序列化队列任务的时候只会保存模型实例的几个关键属性，详细见：`\Illuminate\Contracts\Database\ModelIdentifier`

 
#### 扩展：

1、监控 redis：

```bash
redis-cli > monitor
```

![x](/images/queue/6.png)

2、关于 `Laravel` 队列基本工作方式：`dispatch` 一个 job 的时候，`Laravel` 把 job 序列化保存到相应的 `driver` 中（redis、database、file...），然后 `queue:listen` 或 `queue:work` 的时候会从对应的 `driver` 里面取出这个 job，对 payload 反序列化，然后调用 job 里面的 handle 方法进行 job 的处理。

