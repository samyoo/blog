---
title: git合并多个commit
date: 2018-03-06 16:14:39
tags: [git,rebase]
categories: git
---


>在使用 Git 作为版本控制的时候，我们可能会由于各种各样的原因提交了许多临时的 commit，而这些 commit 拼接起来才是完整的任务。那么我们为了避免太多的 commit 而造成版本控制的混乱，通常我们推荐将这些 commit 合并成一个。


首先假设我们有3个 commit
```
commit 3a78357cc04084a650922453c467d6da9fc77eb2
Author: sam 
Date:   Tue Mar 6 10:38:00 2018 +0800

    commit1

commit 3265065072c9c3f760b260c75b4a99187ea7a244
Author: sam 
Date:   Mon Mar 5 18:19:04 2018 +0800

    commit2

commit 0d2ff3ac759d10b7fad87f09fdd2bb938eadf190
Author: sam 
Date:   Fri Mar 2 17:57:07 2018 +0800

    commit3

```

运行`git rebase -i 0d2ff3ac`其中，-i 的参数是不需要合并的 commit 的 hash 值，这里指的是第一条 commit， 接着我们就进入到 vi 的编辑模式。

```
pick 3a78357c commit1 
pick 32650650 commit2 
 
# Rebase 3a78357c..32650650 onto 0d2ff3ac (3 commands) 
# 
# Commands: 
# p, pick = use commit 
# r, reword = use commit, but edit the commit message 
# e, edit = use commit, but stop for amending 
# s, squash = use commit, but meld into previous commit 
# f, fixup = like "squash", but discard this commit's log message 
# x, exec = run command (the rest of the line) using shell 
# d, drop = remove commit 
# 
# These lines can be re-ordered; they are executed from top to bottom. 
# 
# If you remove a line here THAT COMMIT WILL BE LOST. 
# 
# However, if you remove everything, the rebase will be aborted. 
# 
# Note that empty commits are commented out


```


我们只要知道 pick 和 squash 这两个参数即可。

- pick 的意思是要会执行这个 commit
- squash 的意思是这个 commit 会被合并到前一个commit

我们将 hash1 这个 commit 前方的命令改成 squash 或 s，然后输入:wq以保存并退出


***注意事项：*** 如果这个过程中有操作错误，可以使用 `git rebase --abort`来撤销修改，回到没有开始操作合并之前的状态