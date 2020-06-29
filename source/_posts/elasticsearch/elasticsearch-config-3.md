---
title: Elasticsearch 配置（三）- 重要的系统配置
date: 2020-06-29 11:47:00
tags: [ElasticSearch]
---

理想情况下，Elasticsearch 应该在服务器上单独运行并使用所有可用资源。为此，你需要配置你的系统，以允许运行 Elasticsearch 的用户访问比默认允许更多的资源。

进入生产之前，必须考虑以下设置：

* 禁用交换分区
* 增加文件描述符
* 确保有足够的虚拟内存
* 确保足够的线程
* JVM DNS 缓存设置
* 临时目录未安装 `noexec`


### 开发模式与生产模式

默认情况下，Elasticsearch 假定你正在开发模式下工作。如果以上任何设置的配置都不正确，将在日志文件中写入警告，但是你将能够启动和运行 Elasticsearch 节点。

一旦你配置了像这样的网络设置 `network.host`，Elasticsearch 就会假设你即将投入生产，并将上述警告升级为异常。这些异常将阻止你的 Elasticsearch 节点启动。这是一项重要的安全措施，以确保不会因服务器配置错误而丢失数据。


### 配置系统设置

在哪里配置系统设置取决于您用于安装 Elasticsearch 的软件包以及所使用的操作系统。

使用 `.zip` 或 `.tar.gz` 软件包时，可以配置系统设置：

* 临时配置使用 [ulimit](https://www.elastic.co/guide/en/elasticsearch/reference/master/setting-system-settings.html#ulimit)
* 永久配置在 [/etc/security/limits.conf](https://www.elastic.co/guide/en/elasticsearch/reference/master/setting-system-settings.html#limits.conf)

当使用 RPM 或 Debian 软件包，大部分系统设置在 [系统配置文件](https://www.elastic.co/guide/en/elasticsearch/reference/master/setting-system-settings.html#sysconfig)。然而，使用 `systemd` 的系统要求在 [systemd配置文件](https://www.elastic.co/guide/en/elasticsearch/reference/master/setting-system-settings.html#systemd) 中指定系统限制。


#### `ulimit`

在 Linux 系统上，`ulimit` 可用于临时更改资源限制。通常需要像 `root` 切换到将要运行 Elasticsearch 的用户之前那样设置限制。例如，要将打开的文件句柄（`ulimit -n`）的数量设置为 65536，可以执行以下操作：

```
sudo su
ulimit -n 65535
su elasticsearch
```

1. 切换到 `root` 用户
2. 修改打开文件的最大数量
3. 切换为 `elasticsearch` 用户以启动 Elasticsearch

新限制仅适用于当前会话。

你可以通过查阅所有当前应用的限制 `ulimit -a`。


#### `/etc/security/limits.conf`

在 Linux 系统上，可以通过编辑 `/etc/security/limits.conf` 文件来为特定用户设置永久限制。要将 `elasticsearch` 用户打开的最大文件数设置为 65535，请在 `limits.conf` 文件中添加以下行：

```
elasticsarch - nofile 65535
```

此更改仅在 `elasticsearch` 用户下次打开新会话时生效。


#### Sysconfig 文件

使用 RPM 或 Debian 软件包时，可以在系统配置文件中指定环境变量，该文件位于：

* RPM: `/etc/sysconfig/elasticsearch`
* Debian: `/etc/default/elasticsearch`

但是，需要通过更改 systemd 指定系统限制。


#### 系统配置

在使用 systemd 的系统上使用 RPM 或 Debian 软件包时，必须通过 systemc 指定系统限制。

systemd 服务文件（`/usr/lib/systemd/system/elasticsearch.service`）包含默认情况下应用的限制。

要覆盖它们，请添加一个名为 `/etc/systemd/system/elasticsearch.service.d/override.conf`（或者，你可以运行 `sudo systemctl edit elasticsearch` 编辑该文件），设置该文件中的所有更改：

```
[Service]
LimitMEMLOCK=infinity
```

一旦修改完，运行下面的命令：

```
sudo systemctl daemon-reload
```


### 禁用交换分区

大多数操作系统尝试为文件系统高速缓存使用尽可能多的内存，并急切换出未使用的应用程序内存。这可能会导致部分 JVM 堆设置其可执行页面换出到磁盘上。

交换对性能，节点稳定性非常不利，应不惜一切代价避免交换。它可能会导致垃圾收集持续数分钟而不是毫秒，并且可能导致节点响应缓慢甚至断开与集群的连接。在弹性分布式系统中，让操作系统杀死该节点更为有效。

有三种禁用交换的方法。首选选项是完全禁用交换。如果这不是一个选择，则是否要减少交换性而不是内存锁定取决于你的环境。


#### 禁用所有交换文件

通常，Elasticsearch 是在盒子上运行的唯一服务，其内存使用量由 JVM 选项控制。无需启用交换功能。

在 Linux 系统上，可以通过运行以下命令暂时禁用交换：

```
sudo swapoff -a
```

这不需要重启 Elasticsearch。

要永久禁用它，你将需要编辑 `/etc/fstab` 文件并注释掉所有包含单词 `swap` 的行。


#### 配置 `swappiness`

Linux 系统上可用的另一个选项是确保 `sysctl` 值 `vm.swappiness` 设置为 1。这减少了内核的交换趋势，并且在正常情况下不应导致交换，同时仍然允许整个系统在紧急情况下进行交换。


#### 启用 `bootstrap.memory_lock`

另一种选择是在 Linux/Unix 系统上使用 mlockall 防止任何 Elasticsearch 内存被换出。

> 如果 mlockall 尝试分配的内存超过可用内存，则可能导致 JVM 或 Shell 会话退出。


### 文件描述符

> 这仅与 Linux 和 macOS 有关，如果在 Windows 上运行 Elasticsearch，则可以安全地忽略它。

Elasticsearch 使用许多文件描述符或文件句柄。文件描述符用尽是灾难性的，很可能导致数据丢失。确保将运行 Elasticsearch 的用户的打开文件描述符数限制增加到 65535 或更高。


### 虚拟内存

Elasticsearch `mmapfs` 默认使用目录来存储其索引。默认的操作系统对 mmap 计数的限制可能太低，这可能会导致内存不足异常。

在 Linux 上，你可以通过以下命令来增加限制：

```
sysctl -w vm.max_map_count=262144
```

要永久设置此值，请更新 `/etc/sysctl.conf` 里的 `vm.max_map_count` 设置。要在重启后进行验证，请运行 `sysctl vm.max_map_count`。

RPM 和 Debian 软件包将自动配置此设置。不需要进一步配置。


### 线程数

Elasticsearch 对不同类型的操作使用许多线程池。能够在需要时创建新线程很重要。确保 Elasticsearch 用户可以创建的线程数至少为 4096。

这可以通过 `ulimit -u 4096` 在启动 Elasticsearch 之前设置，或者设置 `/etc/security/limits.conf` 里面的 `nproc` 为 4096。
