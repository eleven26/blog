---
title: vagrant 使用独立下载的 homestead box 文件
date: 2020-06-19 20:36:00
tags: [vagrant, Homestead]
---

1、获取下载链接

命令行直接下载会非常慢，这个只有依靠代理去解决了，我们可以从 vagrant box add 的命令输出中拿到 box 的下载链接，
然后去浏览器通过代理来下载。

2、下载之后，新建一个 metadata.json 文件，内容如下：

```
{
    "name": "laravel/homestead",
    "versions": [{
        "version": "9.5.0",
        "providers": [{
            "name": "virtualbox",
            "url": "file://C:/Users/ruby/Downloads/homestead.box"
        }]
    }]
}
```

这里的 `C:/Users/ruby/Downloads/homestead.box` 是下载的 box 保存的路径。

这里的版本号是我们 homestead up 的时候获取到的版本。

3、最后一步

```
homestead up
```
