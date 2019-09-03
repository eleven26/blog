---
title: apple script 范例
date: 2019-09-03 15:30:00
tags: [apple-script]
---

基本格式 

```AppleScript
tell application <application name> to <verb> <do something>
```

可用动词

* `get` 用来访问窗口属性

* `set` 用来设置窗口属性

* `open` 打开窗口

* `close` 关闭窗口

* `select` 把选中窗口置顶 


1. 获取 Finder 最顶层窗口的标题

```AppleScript
tell application "Finder" to get the name of the front Finder window
```

2. 关闭 Finder 顶层窗口（其他应用类似）

```AppleScript
tell application "Finder" to close the front window
```

3. 打开用户主目录

```
tell application "Finder" to open home
```

4. 打开启动盘

```
tell application "Finder" to open the startup disk
```

5. 获取 Finder 窗口的索引

当打开了多个 Finder 窗口的时候，可以通过索引来获取对应的 Finder 窗口

```
tell application "Finder" to get index of home
```

这里假设是 1，下面的命令将会获取用户主目录窗口的标题

```
tell application "Finder" to get the name of window 1
```

6. 获取第一个窗口的索引

```
tell application "Finder" to get the index of the first window -- 1
```

或

```
tell application "Finder" to get the index of the 1st window
```


7. 获取第二个窗口的索引

```
tell application "Finder" to get the index of the second window -- 2
```

或

```
tell application "Finder" to get the index of the 2nd window
```

8. 根据相对位置获取窗口索引

```
tell application "Finder" to get the front window
--> returns: 1
tell application "Finder" to get the back window
--> returns: 2
tell application "Finder" to get the last window
--> returns: 2
tell application "Finder" to get the index of the Finder window before the last Finder window
--> returns: 1
tell application "Finder" to get the index of the Finder window after the front Finder window
--> returns: 2
```

9. 设置窗口索引（改变窗口层叠顺序）

```
tell application "Finder" to set the index of the last window to 1 -- 最底层窗口置顶
```


### 获取窗口索引方法

* 通过窗口名称

```
by name:
    Finder window "Documents"
by numeric index:
    Finder window 1
by descriptive index:
    the first Finder window
    the second Finder window
    the fifth Finder window
    the 1st Finder window
    the 23rd Finder window
by relative position index:
    the front Finder window
    the middle Finder window
    the back Finder window
    the last Finder window
by random index:
    some Finder window
```


10. 设置 Finder 顶层窗口到某一个目录

这里是 ~/Code

```
tell application "Finder" to set the target of the front Finder window to folder "Code" of home
```

```
tell application "Finder" to set the target of the front Finder window to the startup disk
```

多级目录

```
tell application "Finder" to set the target of the front Finder window to folder "Smith Project" of folder "Documents" of home
```

11. 显示、隐藏 Finder 工具栏

```
tell application "Finder" to set toolbar visible of the front window to false -- 隐藏
```

```
tell application "Finder" to set toolbar visible of the front window to true -- 显示
```

12. 显示、隐藏 Finder 状态栏

```
tell application "Finder" to set statusbar visible of Finder window 1 to true -- 显示
tell application "Finder" to set statusbar visible of Finder window 1 to false -- 隐藏
```

13. 设置侧边栏宽度

```
tell application "Finder" to set the sidebar width of Finder window 1 to 200 -- 设置顶层窗口
tell application "Finder" to set the sidebar width of the second Finder window to 240 -- 设置第二个窗口
tell application "Finder" to set the sidebar width of every Finder window to 0 -- 隐藏所有 Finder 窗口的侧边栏
```

14. Finder view 属性设置

```
tell application "Finder" to get the current view of the front Finder window -- 获取当前 view 属性 (list view, column view, flow view, icon view)

tell application "Finder" to set the current view of the front Finder window to list view

tell application "Finder" to set the current view of the front Finder window to column view

tell application "Finder" to set the current view of the front Finder window to flow view
--> flow view is new in Mac OS X v10.5 (Leopard)

tell application "Finder" to set the current view of the front Finder window to icon view
```

15. 获取顶部 Finder 窗口的位置

```
tell application "Finder" to get the position of the front window
```

16. 设置 Finder 窗口的位置

```
tell application "Finder" to set the position of the front window to {0, 300}
```

17. 获取窗口边界距离屏幕边缘的距离

```
tell application "Finder" to get the bounds of the front window
--> returns something like: {72, 90, 512, 481}
```

各数字意义:

72: 距离屏幕左边边缘的距离

90: 距离屏幕顶部边缘的距离

512: 距离屏幕右边边缘的距离

481: 距离屏幕底部边缘的距离

 
18. 置顶窗口

```
tell application "Finder" to select the last Finder window -- 置顶最底部窗口
```

```
tell application "Finder" to set the index of the last Finder window to 1
```

19. tell 语句块

不用每一个语句都写 `tell application "xxx"`

```
tell application "Finder"
  close every window
  open home
  set toolbar visible of the front Finder window to true
  set the sidebar width of the front window to 135
  set the current view of the front window to column view
  set the bounds of the front Finder window to {36, 116, 511, 674}
  open folder "Documents" of home
  set toolbar visible of the front Finder window to false
  set the current view of the front Finder window to flow view
  set the bounds of front Finder window to {528, 116, 1016, 674}
  select the last Finder window
end tell
```

20. 嵌套 tell 语句块

在对某一个窗口对象操作的时候，可以独立写一个 tell 语句块

```
tell application "Finder"
  close every window

  open home
  tell the front Finder window
    set toolbar visible to true
    set the sidebar width to 135
    set the current view to column view
    set the bounds to {36, 116, 511, 674}
  end tell

  open folder "Documents" of home
  tell the front Finder window
    set toolbar visible to false
    set the current view to flow view
    set the bounds to {528, 116, 1016, 674}
  end tell

  select the last Finder window
end tell
```

21. 激活窗口

```
tell application "Wechat"
    activate
end tell
```

22. 置顶

```
tell application "Wechat"
    set frontmost to true
end tell
```

22. 取消最小化

```
tell application "Wechat"
    try
        set miniaturized of windows to false
    end try
end tell
```
