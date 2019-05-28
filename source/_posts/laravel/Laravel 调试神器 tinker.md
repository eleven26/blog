---
title: Laravel 调试神器 tinker
date: 2018-03-16 22:35:00
tags: [Laravel, PHP]
---

一直以来，想调试框架中的某些东西，如想知道 `Elpquent` 的 `create` 方法返回值是个什么东西，

以前的话，应该就是在 `create` 方法调用之后，使用 `dd` 或者 `var_dump` 之类的函数打印出来

如：
```php
public function getTest()
{
    $user = \App\User::create(['name' => 'tinker']);
    dd($user);
}
```

这样一来，这个流程似乎有点冗长，因为我们还要打开浏览器查看

有了 `tinker`，我们就可以直接在命令行运行我们想要测试的代码，如：


![tinker](/images/8.png)

我们可以看到，`create` 方法返回了一个模型对象。

我们就把这个 `tinker` 当作可以交互式的 代码执行工具好了，我们在学习 `Laravel` 的过程中可以在这里直接测试 `Laravel` 框架中的一些用法，也可以进行调试什么的。
