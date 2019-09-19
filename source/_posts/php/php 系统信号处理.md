---
title: php 系统信号处理
date: 2019-09-19 15:39:00
tags: [PHP]
---


## 依赖 

`pcntl` 扩展


## 源码

signal.php

```php
<?php

declare(ticks=1);

pcntl_signal(SIGUSR1, function () {
    file_put_contents('signal_test.log', time() . PHP_EOL, FILE_APPEND);
});

while (true) {
    sleep(1);
}

```

## 运行

### 查看进程 ID

```
ps aux | grep signal.php | grep -v grep
```

### 发送信号

假设进程 ID 是 12345

```
kill -SIGUSR1 12345
```


## 用处

可以让我们不重启 php 进程的同时做一些处理。

比如，接收到信号后从磁盘读取新的配置文件等。


