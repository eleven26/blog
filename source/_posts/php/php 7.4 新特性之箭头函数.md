---
title: php 7.4 新特性之箭头函数
date: 2019-09-06 20:39:00
tags: [PHP]
---

## 以往写法

在以往的 PHP 版本中，我们使用匿名函数的方式可能会显得非常冗长，即使里面只是一个简单的返回，或者一个简单的表达式。

比如：

```php
$students = [
    [
        'name' => 'Foo',
        'age' => 20,
    ],
    [
        'name' => 'Bar',
        'age' => 21,
    ],
];

$names = array_map(function ($student) {
    return $student['name'];
}, $students);
```

写过 ES6 或者 Java8 的朋友应该对箭头函数这种写法比较熟悉，对比一下：

### ES6

```javascript
let names = students.map(student => student.name)
```

### JAVA8 则类似下面的 map 那样

```java
students.stream()
        .map(student -> student.name)
        .collect(Collectors.toList())
```

相对比之下，php 就显得有些啰嗦，如果闭包函数里面需要使用外部变量，则还需要把每一个外部变量写到 use 里面，那就会更加冗长。


## 箭头函数 hello world

现在我们使用箭头函数来重写一下上面的 `array_map` 调用：

```php
$names = fn ($student) => $student['name'];
```

怎么样？是不是简洁又清晰，看起来特别舒服。 `function`、`return`、`use` 这些全都不用写了。

同时，如果需要使用外部的变量，再也不需要使用 use 来引入那些外部变量了，在箭头函数后面的表达式直接可以使用：

```php
$y = 10;
$fn = fn ($x) => $x + $y;
var_dump($fn(20)); // 30
```

旧的写法：

```php
$y = 10;
$fn = function ($x) use ($y) {
    return $x + $y;
};
var_dump($fn(20)); // 30
```


## 箭头函数写法

### 基本形式

```php
fn (parameter_list) => expr
```

简单说一下（但是很重要）：

* `fn` 是箭头函数标识，原来的 `function` 关键字在箭头函数里只需写 `fn`
* `(parameter_list)` 是箭头函数的参数列表，`fn` 和 括号不能省略
* `expr` 这个名字已经说明了一切，这里我们只能写一个表达式，不能写多个语句，不接受语句块。
* `expr` 是一个表达式，同时也是这个箭头函数的返回值，原来的 `return` 不用写了

有人会说，为什么还要写 `fn`，直接 `parameter => expr` 不就好了吗？

这是因为这种写法已经用于 PHP 的关联数组里面了，冲突了。

想了解为什么的可以看 [rfc/arrow_functions_v2](https://wiki.php.net/rfc/arrow_functions_v2)。


### 函数签名

* `fn (array $x) => $x;`，参数类型限定
* `fn (): int => $x;`，返回值类型限定
* `fn ($x = 42) =>  $x;`，参数默认值
* `fn (&$x) =>  $x;`，参数引用传递
* `fn &($x) =>  $x;`，返回值引用
* `fn ($x, ...$rest) =>  $rest;`

总的来说：

参数传递、参数类型限定、返回值类型限定、引用参数传递、返回值引用等写法和旧的写法一致。

不一样的是：

* 没有了 `function`、`return`、`use` 等关键字，也没有了大括号

* 需要注意的是：不支持大括号，不像 ES6
    - 错误写法：`fn (array $x) => {return $x;};`


## 兼容性问题

* 从 PHP 7.4 起，因为引入了 fn 关键字作为箭头函数的语法，因此，如果你的代码里面有 fn 命名的函数，可能会有冲突。


## 总结

* 基本语法 `fn (parameter_list) => expr`

* PHP7.4 的箭头函数只支持简单的表达式（只能写一个表达式），不能使用大括号把表达式括起来


想尝鲜的可以在 [php74-playground](https://php74-playground.baiguiren.com) 尝试一下！基于 `PHP 7.4.0RC1` 版本。

