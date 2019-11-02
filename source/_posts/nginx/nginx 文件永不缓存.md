---
title: nginx 文件永不缓存
date: 2019-09-10 13:49:00
tags: [linux, nginx]
---

参考链接 : [http://nginx.org/en/docs/http/ngx_http_headers_module.html](http://nginx.org/en/docs/http/ngx_http_headers_module.html)

假设我们要禁用 *.xls 和 *.xlsx 后缀的浏览器缓存：

只需要设置 `expires -1;` 即可

```
location ~ .*\.(xls|xlsx)$
{
    expires      -1;
}
```
