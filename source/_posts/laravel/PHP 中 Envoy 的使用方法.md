---
title: php 中 Envoy 的使用方法
date: 2018-08-01 14:19:00
tags: [Laravel, PHP]
---

##### `envoy` 是什么？

`envoy` 是一个支持 `blade` 语法的 `ssh` 远程命令执行的工具。具体来说就是，通过配置 `ssh` 的账号、密码、`key` 这些，然后可以使用 `envoy` 运行一些预定义的命令（比如 `git` 更新什么的）。

##### 安装
      
```bash
composer global require laravel/envoy
```

##### 怎么使用？

配置（`~/.ssh/config`）：

关于 `ssh key` 配置官网文档没有说，但是 Google 出来了，需要在 `~/.ssh/config` 下面定义你每个 `server` 对应的 `key file`，我们假设里面有如下内容：

```bash
Host 12.34.56.78
    IdentityFile ~/.ssh/id_rsa
    User root
```

如果我们有另外一个服务器，那么可以按如上格式再另写三行，以此类推。

配置有了，接下来就是写具体命令了：

在项目根目录新建一个 `Envoy.blade.php` 文件，这个官网文档有说，第一行定义 `servers`，注意，不能换行，否则会报错，也许是个 bug，如：

```php
@servers(['web' => '127.0.0.1', 'test' => '12.34.56.78'])
```

我们假设有一个更新项目的操作，如：

```php
@task('update', ['on' => ['web', 'test']])
    cd "/path/to/project"
    {{PHP_BINARY}} artisan config:cache
    {{PHP_BINARY}} artisan route:cache
    {{PHP_BINARY}} artisan optimize --force
    "/composer/binary" dumpautoload
@endtask
```

这样我们运行下面命令的时候，就会进行一些操作：

```bash
envoy run update
```

我们可以使用一些 `blade` 的语法，如，在 task 中使用变量，这个变量可以是我们在 `@setup` 中设置的。

tips：对于另外一些操作，比如需要交互的，如可能 `git pull` 的时候需要输入账号密码的，可以尝试一下 `except`（当然，不支持 windows）。

