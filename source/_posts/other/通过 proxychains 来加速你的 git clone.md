---
title: 通过 proxychains 来加速你的 git clone
date: 2020-04-14 13:34:00
tags: [proxychains, proxychains-ng, ShadowSocks]
---

> 前提：有 ShadowSocks

## 安装步骤

1. 下载最新版的 [proxychains-ng](https://github.com/rofl0r/proxychains-ng/releases)

2. 编译安装，详细可以看官方 [readme](https://github.com/rofl0r/proxychains-ng)

```
./configure --prefix=/usr --sysconfdir=/etc
make
sudo make install
sudo make install-config
```

3. 修改 proxychains 配置文件

vim /etc/proxychains.conf

```
# 把最后一行改成下面的，1086 改成本地的 sock5 监听的端口
socks5  0.0.0.0 1086
```

4. 测试

```
proxychains4 telnet google.com 80
```

## 其他可以尝试一下的方式

[git-clone-works-with-https-but-not-ssh-when-using-proxy](https://stackoverflow.com/questions/59384945/git-clone-works-with-https-but-not-ssh-when-using-proxy)
