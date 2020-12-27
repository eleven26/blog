---
title: dcat-admin 中 HasBuilderEvents 的作用
date: 2020-12-27 17:11:02
tags: [dcat-admin]
---

[dcat-admin](https://github.com/jqhph/dcat-admin) 中的 `Form`, `Show`, `Grid`, `Column` 等内置类都 use 了 `Dcat\Admin\Traits\HasBuilderEvents` 这个 trait。

通过这个 trait，我们可以对 `Form`, `Show`, `Grid`, `Column` 的生命周期做一些类似勾子的操作。

`resolving` 主要用在统一自定义某些页面组件是否展示，`composing` 事件用于页面渲染时候做一些额外的操作。常用的主要是 `resolving`。

## resolving

```PHP
// 默认禁用表单的一些 checkbox 以及 header
Form::resolving(function (Form $form) {
    $form->disableHeader();
    $form->disableViewCheck();
    $form->disableEditingCheck();
    $form->disableCreatingCheck();
});
```

## composing

```PHP
$form->composing(function ($form) {
    static::addScript($form);
});
```

```PHP
protected static function addScript(Form $form)
{
    $confirm = json_encode($form->builder()->confirm);

    Admin::script(
        <<<JS
Dcat.FormConfirm = {$confirm};
JS
    );
}
```

页面渲染的时候，添加一段自定义的 js 代码。
