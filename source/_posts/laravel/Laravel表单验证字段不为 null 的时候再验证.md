---
title: Laravel表单验证字段不为 null 的时候再验证
date: 2021-01-12 18:53:00
tags: [Laravel, PHP]
---

在 Laravel 表单验证中我们常常会需要对一些非必填的字段进行校验，比如：`string`，

但是如果如果 data 里面的对应字段是 null，则验证会失败，比如：

```PHP
$data = [
    'num' => null
];

/** @var \Illuminate\Validation\Validator $validator */
$validator = app('validator')->make(
    $data,
    [
        'num' => 'numeric'
    ]
);

if ($validator->fails()) {
    dd($validator->messages()->toArray());
}
```

结果：

```
array:1 [
  "num" => array:1 [
    0 => "num字段值必须为数字"
  ]
]
```


## 如何让其通过验证？

如果我们的这个字段允许为 null 的话，可以加上 nullable 验证规则，比如：

```PHP
$rules = [
    'num' => 'nullable|numeric'
]
```

当然如果保存到数据库的时候不能为 null 的话，就不能这么写了。

