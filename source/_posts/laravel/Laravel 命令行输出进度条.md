---
title: Laravel 命令行输出进度条
date: 2018-06-04 17:05:00
tags: [Laravel, PHP]
---

### 输出进度条

有时候我们想在命令行执行一些耗时的命令，我们可以利用 `symfony` 提供的进度条相关的类，来输出一个进度条，显示当前的处理进度。

参考：[Symfony ProgressBar](http://symfony.com/doc/current/components/console/helpers/progressbar.html)

```php
<?php
 
namespace App\Commands;
 
use Illuminate\Console\Command;
use Symfony\Component\Console\Helper\ProgressBar;
use Symfony\Component\Console\Output\ConsoleOutput;
 
class Test extends Command
{
    protected $signature = 'test';
 
    /**
     * Execute the command.
     *
     * @return void
     */
    public function handle()
    {
        //
        $output = new ConsoleOutput();
        $progressBar = new ProgressBar($output, 1000);
        $progressBar->setFormat("   %elapsed:6s%/%estimated:-6s%   内存消耗: %memory:6s%\n%current%/%max% [%bar%] %percent:3s%%");
 
        foreach (range(1, 1000) as $_) {
            usleep(5000);
            $progressBar->advance();
        }
 
        $progressBar->finish();
        echo "\n";
    }
}
```

效果：

![x](/images/9.png)

输出多个进度条：

```php
$progressBar->start();
print "\n";
$progressBar->start();
```

中间需要打印一个换行符


### 输出动态变化的变量
有时候我们想在命令执行过程中显示一个动态变化的数，同时又想输出的格式保持不变，我们可以使用 `setMessage` 方法
```php
$progressBar->setFormat("已处理数量: %count%");  // 这里是一个占位符，可以和进度条写在一起
$progressBar->setMessage($count, 'count'); // 使用 `$count` 替换输出内容中的 "%$count%"
```

具体效果类似：
```
已处理数量: 1 （这里的 1 会变更，而不是另起一行输出）
```
