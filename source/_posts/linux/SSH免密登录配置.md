---
title: SSH免密登录配置
date: 2018-02-01 14:29:44
tags: [linux]
categories: 运维
---

### 配置SSH无密码登录需要3步：

#### 生成公钥和私钥

```shell
ssh-keygen -t rsa
```
　
默认在` ~/.ssh`目录生成两个文件：`id_rsa`(私钥)、`id_rsa.pub`(公钥)

#### 导入公钥到认证文件,更改权限

```shell
# 导入本机
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

# 导入要免密码登录的服务器,首先将公钥复制到服务器
scp ~/.ssh/id_rsa.pub xxx@host:/home/xxx/id_rsa.pub

# 然后，将公钥导入到认证文件，这一步的操作在服务器上进行
cat ~/id_rsa.pub >> ~/.ssh/authorized_keys

# 在服务器上更改权限
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

#### 测试，第一次登录可能需要`yes`确认，之后就可以直接登录了。

```shell
ssh sam@192.168.1.2
```