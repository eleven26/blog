---
title: nginx 使用 lua 实现动态 document root
date: 2019-08-19 09:41:00
tags: [nginx]
---

### 依赖

* [openresty](https://openresty.org/en/)


### 项目路径

* 根路径： `/usr/local/share/openresty-docroot`
* 该目录下包含多个子目录，运行时的 document root 为其中的某一个目录，具体哪一个根据 HTTP 请求头 Accept 决定
* 默认为 `default` 目录


### nginx 配置

```
worker_processes  1;
error_log logs/error.log;
events {
    worker_connections 1024;
}
http {
    server {
        listen 8081;
        set $docroot "";
        root /usr/local/share/openresty-docroot/$docroot/;
        error_log "/var/log/nginx.log";

        rewrite_by_lua_block {
            ngx.var.docroot = 'default'
            local accept = ngx.req.get_headers()["Accept"]

            local version = string.gsub(accept, 'application/prs.gbcloud.', '')
            version = string.gsub(version, '+json', '')
            if (version ~= accept)
            then
                ngx.var.docroot = version
            end
        }
    }
}
```


### 测试

* 假设 `/usr/local/share/openresty-docroot` 下面有 `default`、`v1.2.3`、`v1.2.4` 三个目录，每个目录包含一个 `index.html` 文件，里面的内容分别是 `default`、`v1.2.3`、`v1.2.4`


### 目录结构

>/usr/local/share/openresty-docroot
├── default
│   └── index.html
├── v1.2.3
│   └── index.html
└── v1.2.4
    └── index.html


### 测试结果
 
* 在 postman 里面不传递 Accept 头的时候，访问 `http://localhost:8081` 会得到 `default`
* 传递 `Accept: application/prs.gbcloud.v1.2.3+json` 的时候，会得到 `v1.2.3`
* 传递 `Accept: application/prs.gbcloud.v1.2.4+json` 的时候，会得到 `v1.2.4`
