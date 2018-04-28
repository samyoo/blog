---
title: fabric管理多台服务器
date: 2018-02-01 14:29:44
tags: [linux,python]
categories: 运维
---

```
fab -f fab.py -u root -p root myip
```

脚本如下:
```
#!/usr/bin/env python
# encoding: utf-8
from fabric.api import local,cd,run,env
env.hosts=['esr@192.168.1.3','esr@192.168.1.4'] #ssh要用到的参数
env.password = '123456'

def update():
    with lcd('/home/sam/IdeaProjects/project ') :
         put(' project  ~/')

def myip():
    with cd('~/'):
        run('./ip.sh')
    #run(" curl icanhazip.com | awk '\''{ print strftime(\"%Y-%m-%d %H:%M:%S\",systime()) \"\t\" $0 } '\'' | tee -a ~/ip.txt ")

def cat():
    run(' cat ~/ip.txt | tail -12')


def clean():
    run("echo ''>~/ip.txt ")



#local('pwd')                     -- 执行本地命令
#lcd('/tmp')                      -- 切换本地目录
#cd('/tmp')                       -- 切换远程目录
#run('uname -a')                  -- 执行远程命令
#sudo('/etc/init.d/nginx start')  -- 执行远程sudo，注意pty选项

```
