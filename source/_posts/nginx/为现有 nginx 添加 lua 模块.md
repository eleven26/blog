---
title: 为现有 nginx 添加 lua 模块
date: 2019-08-27 12:53:00
tags: [nginx]
---

### 安装 LuaJIT

```bash
wget -c http://luajit.org/download/LuaJIT-2.0.5.tar.gz
tar -xzvf LuaJIT-2.0.5.tar.gz
make install PREFIX=/usr/local/LuaJIT
```

然后把下面这一行加到 ~/.bash_profile 或 ~/.zshrc

```bash
export LD_LIBRARY_PATH=/usr/local/LuaJIT/lib:$LD_LIBRARY_PATH
```


### 下载 nginx

```bash
nginx -v  # 假设是 1.11.5
wget http://nginx.org/download/nginx-1.11.5.tar.gz
tar -xzvf nginx-1.11.5.tar.gz
```


### 下载 nginx lua 模块

[lua-nginx-module](https://github.com/openresty/lua-nginx-module/releases)

```bash
wget https://github.com/openresty/lua-nginx-module/archive/v0.10.15.tar.gz
tar -xzvf v0.10.15.tar.gz
```


### 编译 nginx lua 模块

```bash
cd nginx-1.11.5
./configure ...(这里是 nginx -V 里的 configure 选项) --add-dynamic-module=../lua-nginx-module-0.10.15
```

编译完成会在 nginx-1.11.5/objs 下面出现 ngx_http_lua_module.so，这个就是编译生成的 lua 模块


### 修改 nginx 配置

在 nginx.conf 开头加上下面这一行:

```bash
load_module /root/nginx-1.11.3/objs/ngx_http_lua_module.so;
```

当然这里的路径需要根据实际情况修改。


### 测试

```bash
nginx -t
```


### 知识点

* nginx -V 可以查看编译时的选项
