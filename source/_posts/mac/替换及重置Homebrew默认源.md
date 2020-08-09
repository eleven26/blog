---
title: 替换及重置Homebrew默认源
date: 2020-08-09 22:56:00
tags: [brew]
---

## 在未安装之前替换

1. 替换 brew 默认源

```
export HOMEBREW_BREW_GIT_REMOTE=https://mirrors.ustc.edu.cn/brew.git
```

2. 替换 homebrew/homebrew-core 默认源（中科大源）：

```
export HOMEBREW_CORE_GIT_REMOTE=https://mirrors.ustc.edu.cn/homebrew-core.git
```

如果想下次启动的时候，还使用这两个源，需要做以下修改：

```
替换brew.git:
cd "$(brew --repo)"
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git

替换homebrew-core.git:
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
```

参考链接：

[https://lug.ustc.edu.cn/wiki/mirrors/help/brew.git](https://lug.ustc.edu.cn/wiki/mirrors/help/brew.git)