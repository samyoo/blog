---
title: systemd配置tomcat开机自启动
date: 2018-02-22 14:29:44
tags: [linux,ubuntu]
categories: 运维
---

### 添加Tomcat启动参数配置
`catalina.sh`在执行的时候会调用同级路径下的`setenv.sh`来设置额外的环境变量，因此需要在`/opt/tomcat/bin`路径下创建`setenv.sh`文件。 
```ini
export JAVA_HOME=/opt/jdk
export JRE_HOME=$JAVA_HOME/jre

export CATALINA_HOME=/opt/tomcat
export CATALINA_BASE=/opt/tomcat
#设置Tomcat的PID文件
CATALINA_PID="$CATALINA_BASE/tomcat.pid"
#添加JVM选项
JAVA_OPTS="-server -XX:PermSize=256M -XX:MaxPermSize=1024m -Xms512M -Xmx1024M -XX:MaxNewSize=256m -Duser.timezone=GMT+08"
```

### 编写tomcat.service文件
在`/usr/lib/systemd/system`路径下添加`tomcat.service`文件。 
**注意：文件中目录路径必须是绝对路径**

```ini
[Unit]
Description=Tomcat8
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/opt/tomcat/tomcat.pid
ExecStart=/opt/tomcat/bin/startup.sh
ExecReload=/opt/tomcat/bin/shutdown.sh
ExecStop=/opt/tomcat/bin/shutdown.sh
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

- `[unit]`配置了服务的描述，规定了在network启动之后执行， 
- `[service]`配置服务的pid，服务的启动，停止，重启 
- `[install]`配置了使用用户

### 测试脚本
```shell

systemctl enable tomcat  # 设置开机自启动
Created symlink from /etc/systemd/system/multi-user.target.wants/tomcat.service   
to /usr/lib/systemd/system/tomcat.service.       # 自动创建软连接

systemctl disable tomcat # 禁用开机启动
systemctl daemon-reload # 刷新配置

systemctl start tomcat  # 启动tomcat服务
systemctl status tomcat # 查看tomcat服务状态
```