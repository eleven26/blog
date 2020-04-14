---
title: brew 使用 ShadowSocks 代理
date: 2020-04-14 13:34:00
tags: [brew, ShadowSocks]
---

```
export ALL_PROXY=socks5://127.0.0.1:1086
```

> 1086 是 ShadowSocks 的 socks5 流量监听端口

然后执行 brew 命令。
