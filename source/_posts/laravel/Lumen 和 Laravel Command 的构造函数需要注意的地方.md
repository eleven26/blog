---
title: Laravel/Lumen Command 的构造函数需要注意的地方
date: 2019-04-14 09:11:00
tags: [Laravel, PHP]
---

比如 `Lumen`，`ConsoleServiceProvider` 里面的 `register` 做了下面的处理：

`\Laravel\Lumen\Console\ConsoleServiceProvider::register`


```php
/**
     * Register the service provider.
     *
     * @return void
     */
    public function register()
    {
        $this->registerCommands(array_merge(
            $this->commands, $this->devCommands
        ));
    }
 
    /**
     * Register the given commands.
     *
     * @param  array  $commands
     * @return void
     */
    protected function registerCommands(array $commands)
    {
        foreach (array_keys($commands) as $command) {
            call_user_func_array([$this, "register{$command}Command"], []);
        }
 
        $this->commands(array_values($commands));
    }
 
    /**
     * Register the package's custom Artisan commands.
     *
     * @param  array|mixed  $commands
     * @return void
     */
    public function commands($commands)
    {
        $commands = is_array($commands) ? $commands : func_get_args();
 
        Artisan::starting(function ($artisan) use ($commands) {
            $artisan->resolveCommands($commands);
        });
    }
 
 
    // \Illuminate\Console\Application::resolveCommands
 
    /**
     * Resolve an array of commands through the application.
     *
     * @param  array|mixed  $commands
     * @return $this
     */
    public function resolveCommands($commands)
    {
        $commands = is_array($commands) ? $commands : func_get_args();
 
        foreach ($commands as $command) {
            $this->resolve($command);
        }
 
        return $this;
    }
 
    /**
     * Add a command, resolving through the application.
     *
     * @param  string  $command
     * @return \Symfony\Component\Console\Command\Command
     */
    public function resolve($command)
    {
        return $this->add($this->laravel->make($command));
    }
```

注意看上面的最后一个方法，这是所有命令 `register` 的时候都要做的一件事，也就是实例化，
也就是说，在我们命令行任何的 `handle` 方法运行之前，就需要实例化所有的 `Command` 了。
因为 `Command` 里面的 `signature`、`description` 都是实例属性，需要实例化才能获取，要不然我们运行 `php artisan` 的时候就获取不到命令列表了。

因此，如果我们在 `Command` 的 `__construct` 里面做了一些影响全局的操作，有可能我们不会察觉得到，比如：

```php
\Cache::setPrefix('abc');
```

虽然你是在一个 `Command` 里面设置了 `Cache` 的 `prefix`，但实际上你在其他 `Command` 里面的缓存前缀也会是你设置的这个。

简单地说，我们运行 `php artisan` 的时候，会把所有定义的 `Command` 实例化，如果我们需要做一些命令行初始化操作，最好放在 `handle()` 里面
