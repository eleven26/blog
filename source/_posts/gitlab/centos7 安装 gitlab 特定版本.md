---
title: centos7 安装 gitlab 特定版本
date: 2019-08-30 10:01:00
tags: [gitlab]
---

[安装指引](https://docs.gitlab.com/omnibus/manual_install.html)

这里假设要安装 gitlab-ce 10.3.3

先在 [https://packages.gitlab.com/gitlab/gitlab-ce](https://packages.gitlab.com/gitlab/gitlab-ce) 找到对应版本的 gitlab-ce

[gitlab-ce-10.3.3-ce.0.el7.x86_64.rpm](https://packages.gitlab.com/gitlab/gitlab-ce/packages/scientific/7/gitlab-ce-10.3.3-ce.0.el7.x86_64.rpm)


### 安装

```bash
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
```

```bash
sudo yum install gitlab-ce-10.3.3-ce.0.el7.x86_64
```

### 启动

```bash
gitlab-ctl start
```


### 配置 gitlab 域名和端口

修改 /etc/gitlab/gitlab.rb，如

```
external_url 'http://gitlab.local:8888'
```

修改配置之后，需要运行：

```bash
gitlab-ctl reconfigure
```


### 查看安装信息

打开 `http://gitlab.local:8888/help`
