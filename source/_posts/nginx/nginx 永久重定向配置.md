---
title: nginx 永久重定向配置.md
date: 2019-08-02 13:57:00
tags: [nginx]
---

```
location ^~ /abc/ {
   return 301 http://192.168.2.123:8503$uri;
}
```

### 含义

* `^~` 正则匹配后面的路径 (这里是 `/abc/`)，这里 location 的作用是匹配所有 uri 前缀是 `/abc/` 的请求

*  `301` 永久重定向

* `$uri` 加上这个防止重定向之后 uri 丢失
