---
title: CentOS 6.10 安装 RabbitMQ
date: 2020-03-02 14:38:00
tags: [linux]
---

参考链接：[https://tecadmin.net/install-rabbitmq-on-centos/](https://tecadmin.net/install-rabbitmq-on-centos/)

## 安装 Erlang

```
wget https://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
sudo rpm -Uvh erlang-solutions-1.0-1.noarch.rpm
sudo yum install erlang erlang-nox
```

## 安装 RabbitMQ Server

CentOS/RHEL 6

```
https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.2/rabbitmq-server-3.8.2-1.el6.noarch.rpm
```

```
sudo rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
sudo yum install rabbitmq-server-3.8.2-1.el6.noarch.rpm
```

## 疑难杂症

在内网测试机上安装的过程中，遇到了一种情况，`yum install` 只能安装旧的版本：

```
yum list | grep erlang

erlang-appmon.x86_64                        R14B-04.3.el6               @epel
erlang-asn1.x86_64                          R14B-04.3.el6               @epel
erlang-common_test.x86_64                   R14B-04.3.el6               @epel
erlang-compiler.x86_64                      R14B-04.3.el6               @epel
erlang-cosEvent.x86_64                      R14B-04.3.el6               @epel
erlang-cosEventDomain.x86_64                R14B-04.3.el6               @epel
erlang-cosFileTransfer.x86_64               R14B-04.3.el6               @epel
erlang-cosNotification.x86_64               R14B-04.3.el6               @epel
erlang-cosProperty.x86_64                   R14B-04.3.el6               @epel
erlang-cosTime.x86_64                       R14B-04.3.el6               @epel
...
```

这个命令输出显示了一堆 R14B-04.3.el6 的 erlang 版本。但是这不是我想要安装的版本。而且因为这一个版本存在，也导致新的版本死活安装不上。

最后搜索 @epel 找到一个命令：

```
rpm -qa | grep kernel
```

这个命令输出显示有一个 `erlang-kernel-R14B-04.3.el6.x86_64`，说实话这个不知道哪里来的，估计是之前随便找了篇文章瞎搞上去的。

```
kernel-2.6.32-642.el6.x86_64
kernel-2.6.32-696.23.1.el6.x86_64
kernel-2.6.32-754.27.1.el6.x86_64
dracut-kernel-004-411.el6.noarch
kernel-2.6.32-754.6.3.el6.x86_64
erlang-kernel-R14B-04.3.el6.x86_64
kernel-headers-2.6.32-754.27.1.el6.x86_64
kernel-firmware-2.6.32-754.27.1.el6.noarch
```

把这个删除就可以装上新的版本了。

```
yum remove erlang-kernel-R14B-04.3.el6.x86_64
```

