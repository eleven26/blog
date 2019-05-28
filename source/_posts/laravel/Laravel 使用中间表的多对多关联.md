---
title: Laravel 使用中间表的多对多关联
date: 2017-12-11 18:04:00
tags: [Laravel, PHP]
---

用户模型 `App\User`

标签模型 `App\Tag`

中间表 `user_tag` (`user_id`, `tag_id`)

在user模型中定义tags关联如下：


```php
class User extend Model
{
    public function tags()
    {
        return $this->belongsToMany('App\Tag', 'user_tag');
    }
}
```

上面的 `belongsToMany` 方法中的第二个参数 `user_tag` 是中间表，保存了 `user_id` 和 `tag_id`。
其中 `user_id` 关联 `users` 表 `id`, `tag_id` 关联 `tags` 表 `id`。
