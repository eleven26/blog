---
title: Git 同步 fork 的项目
date: 2019-03-21 13:53:30
tags: Git
---

```
# Add the remote, call it "upstream":

git remote add upstream https://github.com/whoever/whatever.git

# Fetch all the branches of that remote into remote-tracking branches,
# such as upstream/master:

git fetch upstream

# Make sure that you're on your master branch:

git checkout master

# Rewrite your master branch so that any commits of yours that
# aren't already in upstream/master are replayed on top of that
# other branch:

git rebase upstream/master
```

[查看原文](https://stackoverflow.com/questions/7244321/how-do-i-update-a-github-forked-repository)
