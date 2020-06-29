---
title: Elasticsearch 配置（一）
date: 2020-06-29 11:47:00
tags: [ElasticSearch]
---

Elasticsearch 具有良好的默认设置，并且只需要很少的配置。可以使用 [cluster-update-setting](https://www.elastic.co/guide/en/elasticsearch/reference/master/cluster-update-settings.html) API 在正在运行的集群上更改大多数设置。

配置文件应包含特定于节点的设置（例如 `node.name` 和路径），或节点为了能够加入集群所需的设置，例如 `cluster.name` 和 `network.host`。


### 配置文件位置

Elasticsearch 具有三个配置文件：

* `elasticsearch.yml` 用于配置 Elasticsearch
* `jvm.options` 用于配置 Elasticsearch JVM 设置
* `log4j.properties` 用于配置 Elasticsearch 日志记录

这些文件位于 config 目录中，其默认位置取决于安装是来自自归档发行版（`tar.gz` 或 `zip`）还是软件包发行版（Debian 或 RPM 软件包）。

对于存档分发，配置目录位置默认为 `$ES_HOME/config`。可以通过 `ES_PATH_CONF` 环境变量来更改 config 目录的位置，如下所示：

```
ES_PATH_CONF=/path/to/my/config ./bin/elasticsearch
```

或者，您可以通过命令行或外壳配置文件来 `export` 使用 `ES_PATH_CONF` 环境变量。

对于软件包分发，config 目录位置默认为 `/etc/elasticsearch`。config 目录的位置也可以通过 `ES_PATH_CONF` 环境变量来更改，但是请注意，在你的 shell 中进行设置是不够的。而是，此变量来自 `/etc/default/elasticsearch` (对于 Debian 软件包) 和 `/etc/sysconfig/elasticsearch` (对于 RPM 软件包)。您将需要相应地 `ES_PATH_CONF=/etc/elasticsearch` 在这些文件之一中编辑条目，以更改配置目录位置。


### 配置文件格式

配置格式为 YAML。这是更改数据和日志目录的路径的示例。

```
path:
    data: /var/lib/elasticsearch
    logs: /var/log/elasticsearch
```

设置也可以按照如下方式展平：

```
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
```


### 环境变量替换

`${...}` 在配置文件中用符号引用的环境变量将替换为环境变量的值。例如：

```
node.name: ${HOSTNAME}
network.host: ${ES_NETWORK_HOST}
```

环境变量的值必须是简单的字符串。使用逗号分隔的字符串，Elasticsearch 将解析为列表。例如，Elasticsearch 将以下字符串拆分为 `${HOSTNAME}` 环境变量的值列表：

```
export HOSTNAME="host1, host2"
```


### 集群和节点设置类型

可以根据配置方式对集群和节点进行分类：

#### 动态

你可以使用 [cluster-update-API](https://www.elastic.co/guide/en/elasticsearch/reference/master/cluster-update-settings.html) 在正在运行的集群上配置和更新动态设置。

你还可以使用来在未启动或关闭的节点上本地配置动态设置 `elasticsearch.yml`。

> 最好使用 cluster-update-API 设置动态，集群范围内的设置，并且 `elasticsearch.yml` 仅用于本地配置。使用集群更新 API 可以确保所有节点上的设置都相同。如果您不小心在不同的接点上配置了不同的设置，则可能很难注意到差异。


#### 静态的

只能使用在未启动或关闭的节点上配置静态设置 `elasticsearch.yml`。

必须在集群中的每个相关节点上设置静态设置。
