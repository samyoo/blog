---
title: 开机出现busybox v1.22.1(debian 1:1.22.0-19) built-in shell(ash)解决方案
date: 2018-03-01 09:33:44
tags: [linux,ubuntu]
categories: 运维
---

出现这个问题的根源是硬盘处错误了，再开机logo时按esc就会看到检查错误。下面记录下解决方法。

```shell
#这个是查看硬盘的命令里面会列出所有分区
blkid
#运行fsck命令，注意sdaX，X代表的就是你的分区号，可以在列表中找到
fsck -y /dev/sdaX
#完成后执行exit退出重启电脑就可以了
exit
```

- [`几种引导异常导致无法正常开机的解决办法`](http://www.jianshu.com/p/0cb89814bc08)