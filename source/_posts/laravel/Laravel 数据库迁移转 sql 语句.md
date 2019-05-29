---
title: 可以使用下面的命令
date: 2018-08-02 08:27:00
tags: [Laravel, PHP]
---

可以使用下面的命令

```php
php artisan migrate --pretend --no-ansi　　
```

当然，你需要有可以 migrate 的东西。

数据库迁移导出到文件（使用命令）

```php
<?php
 
namespace App\Console\Commands;
 
use Illuminate\Contracts\Bus\SelfHandling;
 
class MigrateToSql implements SelfHandling
{
    protected $signature = 'migrate_to_sql';
 
    protected $description = '数据库迁移转 sql';
 
    /**
     * Execute the command.
     *
     * @return void
     */
    public function handle()
    {
        $command = PHP_BINARY . ' ' . base_path('artisan') . ' migrate --pretend --no-ansi';
        exec($command, $output);
 
        $sql = '';
        if (count($output) > 0) {
            foreach ($output as $line) {
                $sql .= preg_replace('/.*?:/', '', $line) . ";\n";
            }
        }
 
        $file = database_path('sqls/' . date('Y-m-d H:i:s') . '.sql');
        file_put_contents($file, $sql);
    }
}
```

上面的一些处理是把一些无效的信息去掉，如时间戳，这样最后剩下的就是可以直接执行的 sql 语句了。

