---
title: Private registry push fail server gave HTTP response to HTTPS client
date: 2019-10-23 09:16:30
tags: Docker
---

1. 修改或创建 `/etc/docker/daemon.json`

```json
{ 
  "insecure-registries": ["myregistry.example.com:5000"]
}
```

2. 重启 docker daemon (不同系统不一样)

```
systemctl restart docker
```
