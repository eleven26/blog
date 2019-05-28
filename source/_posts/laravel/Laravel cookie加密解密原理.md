---
title: Laravel cookie加密解密原理
date: 2018-01-11 18:37:00
tags: [Laravel, PHP]
---

通过控制台的 `cookie` 信息我们会发现，每次请求之后，关键的 `cookie`，如`PHPSESSID`、`XSRF-TOKEN` 都会发生变化，并且都是很长的一串字符串。

其实这是一个 `json` 数组，其中包含了 `iv`，`value`，`mac` 三个字段：

![cookie](/images/5.png)

这些字段都是在框架加密解密的时候使用的，加密方法是 [openssl_encrypt](https://php.net/manual/zh/function.openssl-encrypt.php)：

![openssl_encrypt](/images/6.png)

对 `openssl` 不太了解的可以看下下面的例子：

```php
$data = 'laravel';
$iv = random_bytes(16);
$key = 'this is key';
 
$encrypt = openssl_encrypt($data, 'AES-256-CBC', $key, 0, $iv);
 
var_dump($encrypt);
var_dump(openssl_decrypt($encrypt, 'AES-256-CBC', $key, 0, $iv));
```

`Laravel` 中的话，`key` 就是 `.env` 配置文件里面的 `APP_KEY`，除了 `key` 还有两个变化的参数就是 加密、解密的数据以及 `iv`。

也就是说，如果我们需要加密 `cookie` 的话，我们至少得保存下 加密后的数据以及 `iv`。

这样看来，`mac` 字段似乎有点多余，但是我们可以使用该字段来验证数据的合法性：

![x](/images/7.png)

如果验证不通过，`Laravel` 也就不会对 `data` 进行解密操作。

虽然每次请求 `cookie` 都会发生变化，但是实际数据是没有变的，发生变化只是因为用来加密的 `iv` 变了(使用 `random_bytes` 方法生成)。

由于 `iv` 每次都变化，所以需要把 `iv` 也一同返回给浏览器，加上验证数据合法性的 `mac`，最后返回的就是下面的数组的 `json` 编码后在 `base64` 编码的数据：

```php
[
   'iv' => random_bytes(16), // 16位随机字节串
   'value' => 'xxxx...',    // 加密后的数据
   'mac' => 'xxx...'   // 后续请求验证数据合法性的字符串
]
```
