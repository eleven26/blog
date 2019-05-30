---
title: 实用 zsh 插件
date: 2019-01-13 10:19:00
tags: [zsh, oh-my-zsh]
---

`zsh` 命令补全插件

[zsh-users/zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)

![x](/images/zsh/1.png)


`laravel5`（使用前提：安装了 `oh-my-zsh`）

![x](/images/zsh/2.png)

使用方法，修改 `~/.zshrc`，在 `plugins` 里面加一行 `laravel5`，然后运行 `source ~/.zshrc`

主要用处：在输入 `artisan` 的时候可以自动进行命令补全（包括自定义的一些命令）
