---
title: Git同步Fork代码
date: 2018-02-28 11:21:44
tags: [git,fork]
categories: git
---

### 给fork配置远程库
使用
```shell
git remote -v
```

### 查看远程状态
确定一个将被同步给 fork 远程的上游仓库
```shell
git remote add upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git
```
再次查看状态确认是否配置成功。

### 同步fork
从上游仓库 fetch 分支和提交点，提交给本地 master，并会被存储在一个本地分支 upstream/master
```shell
git fetch upstream
```

### 切换到本地主分支(如果不在的话)
```shell
git checkout master
```

### 把 upstream/master 分支合并到本地 master 上，这样就完成了同步，并且不会丢掉本地修改的内容。
```shell
git merge upstream/master
```

### 如果想更新到 GitHub 的 fork 上，直接
```shell
git push origin master
```

就好了。