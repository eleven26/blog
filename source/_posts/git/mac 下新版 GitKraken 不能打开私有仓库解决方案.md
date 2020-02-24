---
title: mac 下新版 GitKraken 不能打开私有仓库解决方案
date: 2020-01-15 08:43:00
tags: [Git, GitKraken, electron]
---

> 本文只是说 mac 下的解决方案，其他系统可以依样画葫芦，毕竟用 electron 就是为了它的跨平台特性，各个平台代码基本上一样的。

今天打开 GitKraken 的时候突然发现，提示我 `The free version of GitKraken does not support opening private or self-hosted repositories.`。

`GitKraken` 是一个用 `electron` 开发的客户端应用，正好之前用 `electron` 写过一点东西，去看看源码有没有什么解决方法。

源码位于：`/Applications/GitKraken.app/Contents/Resources`

```
ls /Applications/GitKraken.app/Contents/Resources
```

我们会看到一个 `app.asar` 文件，这个其实就是应用的核心所在，我们可以看作类似与 zip 之类的东西，它把所有源码打包到了一个文件中，我们解压开来看看源码：

> 想了解 asar 是什么的可以看 [https://www.electronjs.org/docs/tutorial/application-packaging](https://www.electronjs.org/docs/tutorial/application-packaging)

```
cd /Applications/GitKraken.app/Contents/Resources
asar e app.asar unpacked  # 解压 app.asar 到 unpacked 文件夹中
```

asar 需要通过 npm 安装：

```
npm install -g asar
```

我们可以直接用 WebStorm 之类的 IDE 打开这个文件夹，我们可以看到一个 `initializeGlobals.js` 文件，在里面发现一行对于开发人员来说比较敏感的代码：

```
process.env.NODE_ENV = loadSettings.mode === 'development' ? 'development' : 'production';
```

一般开发中，我们常常定义 env 环境变量，然后在开发环境我们通常会少一些限制，想到此，可能修改为 `development` 模式会可以解决现在遇到的问题，然后找到定义的地方，在另外一个文件中：

clientType.js

```
module.exports = 'production';
```

我们把这一行修改为 `module.exports = 'development';` 看看。

修改完之后，重新打包。

1. 先把旧的文件备份：

```
cp /Applications/GitKraken.app/Contents/Resources/app.asar /Applications/GitKraken.app/Contents/Resources/app.asar.bak
```

2. 打包之后，我们就可以将修改后的代码重新打包

```
cd /Applications/GitKraken.app/Contents/Resources
rm app.asar && asar pack unpacked app.asar
```

这里的 `unpacked` 就是刚刚上面解压出来的文件夹。

3. 重启 `GitKraken`

重启之后，弹窗没有了，各项功能正常使用。应该没什么问题了，就先用着了。


参考链接：

1. [application-packaging](https://www.electronjs.org/docs/tutorial/application-packaging)
