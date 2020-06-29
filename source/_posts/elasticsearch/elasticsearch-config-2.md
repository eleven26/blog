---
title: Elasticsearch 配置（二）- 重要的 Elasticsearch 配置
date: 2020-06-29 11:47:04
tags: [ElasticSearch]
---

尽管 Elasticsearch 需要很少的配置，但在投入生产之前，需要考虑许多设置。

进入生产之前，必须考虑以下设置：

* 路径设定
* 集群名称
* 节点名称
* 网络主机
* 发现设置
* 堆大小
* 堆转储路径
* GC 记录
* 临时目录


### `path.data` 和 `path.logs`

如果你使用 `.zip` 或 `.tar.gz` 归档，则 `data` 和 `logs` 目录是 `$ES_HOME` 的子目录。如果这些重要的目录都保留在它们默认的位置，我们很有可能会在升级 Elasticsearch 版本的生活会将其删除。

在生产中使用时，几乎肯定会想要更改 data 和 log 文件夹的位置：

```
path
    logs: /var/log/elasticsearch
    data: /var/data/elasticsearch
```

RPM 和 Debian 发行版已经为 `data` 和 `logs` 使用了自定义路径。

`path.data` 设置可以设置多个路径，在这种情况下，所有的路径将被用于存储数据（虽然属于单个碎片文件将全部存储在相同的数据路径上）：

```
path:
    data:
        - /mnt/elasticsearch_1
        - /mnt/elasticsearch_2
        - /mnt/elasticsearch_3
```


### `cluster.name`

当 `cluster.name` 节点与集群中的所有其他节点共享节点时，该节点只能加入集群。默认名称为 `elasticsearch`，但您应将其更改为描述集群用途的适当名称。

```
cluster.name: logging-prod
```

确保不要在不同的环境中重复使用相同的集群名称，否则可能会导致节点加入了错误的集群。


### `node.name`

Elasticsearch `node.name` 用作 Elasticsearch 特定实例的人类可读标识符，因此它包含在许多 API 的响应中。它默认为计算机在 Elasticsearch 启动时具有的主机名，但可以在 `elasticsearch.yml` 按以下方式显式配置：

```
node.name: prod-data-2
```


### `network.host`

默认情况下，Elasticsearch 仅绑定到环回地址（例如 `127.0.0.1` 和 `[::1]`）。这足以在服务器上运行单个开发节点。

> 实际上，可以从 `$ES_HOME` 单个节点上的相同位置启动多个节点。这对于测试 Elasticsearch 形成集群的能力很有用，但不是建议用于生产的配置。

为了与其他服务器上的节点形成集群，您的节点将需要绑定到非环回地址。尽管网络设置很多，通常你需要配置的是 `network.host`：

```
network.host: 192.168.1.10
```

该 `network.host` 设置也了解一些特殊的值，比如 `_local_`，`_site_`，`_global_` 和诸如 `:ip4`、`:ip6` 这种修饰符。

一旦提供了自定义的 `network.host`，Elasticsearch 就会假设你正在从开发模式过渡到生产模式，并将许多系统启动检查从警告升级为异常。
