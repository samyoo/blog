---
title: gitlab安装配置
date: 2018-02-28 11:45:44
tags: [git,gitlab]
categories: git
---

### 搭建

```shell
#安装依赖软件
sudo apt-get install -y curl openssh-server ca-certificates
sudo apt-get install -y postfix

#Add the GitLab package repository.
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash

#install the GitLab package.
sudo apt-get install gitlab-ce

#Configure and start GitLab
sudo gitlab-ctl reconfigure
```

### 特殊配置

 gitlab安装完成后会有一个默认的存储代码仓库的路径，建议自己自定义到大磁盘上，以免之后磁盘空间引起不必要的麻烦。

#### 配置gitlab的项目代码存储目录为 /data/GitData/git-data/
```shell
mkdir -pv /data/GitData/git-data/
chown git -R  /data/GitData/git-data/
```

#### 修改配置文件

```shell
# /etc/gitlab/gitlab.rb 添加配置：
git_data_dirs({ "default" => { "path" => "/data/GitData/git-data" } })
```
#### 配置访问的域名.

```shell
#/etc/gitlab/gitlab.rb 修改配置：
external_url 'http://git.xxxx.com'
```
#### 配置生效
```shell
gitlab-ctl reconfigure
```

### 使用命令
```shell
#查看状态：
sudo gitlab-ctl status;
 
#Start all GitLab components (启动)
sudo gitlab-ctl start
 
#Stop all GitLab components (停止)
sudo gitlab-ctl stop
 
#Restart all GitLab components(重启)
sudo gitlab-ctl restart
```
### 备份
 使用一条命令即可创建完整的Gitlab备份:
```shell
gitlab-rake gitlab:backup:create
```
使用以上命令会在/var/opt/gitlab/backups目录下创建一个名称类似为`1508136272_2017_10_16_10.0.2_gitlab_backup.tar`的压缩包, 这个压缩包就是Gitlab整个的完整部分, 其中开头的1508136272_2017_10_16是备份创建的日期,10.0.2是gitlab的版本号。

### 恢复
```shell
#停止相关数据连接服务
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq
  
#从1393513186编号备份中恢复
gitlab-rake gitlab:backup:restore BACKUP=1508136272
  
#启动Gitlab
sudo gitlab-ctl start
```

---
参考 : [官网](https://about.gitlab.com/)
