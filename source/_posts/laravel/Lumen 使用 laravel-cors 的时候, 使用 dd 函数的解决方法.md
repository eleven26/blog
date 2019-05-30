---
title: Lumen 使用 laravel-cors 的时候, 使用 dd 函数的解决方法
date: 2018-12-14 22:13:00
tags: [Laravel, PHP]
---

```php
if (! function_exists('dd')) {
    /**
     * Dump the passed variables and end the script.
     *
     * @param  mixed  $args
     * @return void
     */
    function dd(...$args)
    {
        header('Access-Control-Allow-Origin: *');
        header('Access-Control-Allow-Methods: *');
        header('Access-Control-Allow-Headers: *');
        http_response_code(500);
 
        foreach ($args as $x) {
            (new Illuminate\Support\Debug\Dumper)->dump($x);
        }
 
        die(1);
    }
}
```
