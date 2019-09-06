---
title: ubuntu trusty root jail 设置
date: 2019-09-06 09:00:00
tags: [linux]
---

基于 ubuntu 14.04


### 安装 schroot

```bash
apt-get update
apt-get install schroot
```

`schroot` 允许用户在一个 `chroot` 环境中允许命令


### 安装 debootstrap

```
apt-get install debootstrap
```

`debootstrap` 是 Debian 系列系统下安装系统到子目录的一个工具。我们的 root jail 需要这个工具把系统的一些基本的东西安装都 root jail 目录下。


### 创建 root jail 目录

```
mkdir /var/jail
```


### 修改 schroot 配置文件

trusty 是 ubuntu 版本代号，对应是 ubuntu 14.04

```
[trusty]
description=Ubuntu precise
location=/var/jail
priority=3
users=devops
groups=devops
root-groups=root
```

### 安装

```
debootstrap trusty /var/jail/ http://mirrors.163.com/ubuntu/
```
最后的 `http://mirrors.163.com/ubuntu/` 是镜像地址


### 测试

```
chroot /var/jail
```

从 root jail 退出

```
exit
```


### 后续设置

```
mount -o bind /proc /var/jail/proc
```

这个操作可以让我们在 root jail 中运行 ps 等命令


