---
title: Laravel/PHP 一些调试技巧
date: 2019-05-11 11:48:45
tags: [Laravel, PHP]
---

#### 1. 模型属性不知道哪里修改？

直接覆盖模型的 setAttribute 方法，监测到某一个属性改动的时候，抛一个异常就可以看到堆栈了

```php
use Illuminate\Database\Eloquent\Model;

class Person extends Model
{
    public function setAttribute($key, $value)
    {
        if ($key == 'xxx') { // 'xxx' 是你想要监听的属性
            throw new \RuntimeException;
        }

        return parent::setAttribute($key, $value);
    }
}
```

有多个地方修改？抛异常，捕获写 log，然后去 log 里面看。

```php
use Illuminate\Database\Eloquent\Model;

class Person extends Model
{
    public function setAttribute($key, $value)
    {
        if ($key == 'xxx') { // 'xxx' 是你想要监听的属性
            try {
                throw new \RuntimeException;
            } catch (\RuntimeException $e) {
                \Log::error($e->getTraceAsString());
            }
        }

        return parent::setAttribute($key, $value);
    }
}
```

#### 2. Try to get property of non-object ?

```php
$a = null;

var_dump($a->c);
```

 对于这种场景，我们也可以 try catch，然后在异常处理里面 dump 上下文一些关键的变量、对象这些东西。

