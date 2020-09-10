---
title: 使用 cpulimit 限制进程 CPU 占用
date: 2020-09-09 22:00:00
tags: [cpulimit, 效率]
---

因为最近这电脑在 CPU 暴涨的时候会很容易死机，所以需要找个办法来限制一下一些进程的 CPU 占用，防止这些进程运行的时候导致死机。

## 安装 cpulimit

```
wget -O cpulimit.zip https://github.com/opsengine/cpulimit/archive/master.zip
unzip cpulimit.zip
cd cpulimit-master
make
sudo cp src/cpulimit /usr/bin
```

## 使用 cpulimit

```
cpulimit -l 50 -i <这里接你想要执行的命令>
```

比如，我想限制一下 `npm install` 的 cpu 占用，则可以像如下这样使用 cpulimit

```
cpulimit -l 50 -i npm install
```

执行之后，`npm install` 产生的那个进程占用的 CPU 占用率就不会再超过 50% 了。

