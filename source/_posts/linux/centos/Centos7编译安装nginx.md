---
title: Centos7 编译安装nginx
date: 2018-02-24 14:29:44
tags: [linux,centos,nginx]
categories: 运维
---

### 安装依赖包
```shell
#安装make
yum -y install gcc automake autoconf libtool make

#安装g++
yum -y install gcc gcc-c++

#安装网络工具
yum -y install net-tools
```


### 安装PCRE库
```shell
cd /opt
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.39.tar.gz
tar -zxvf pcre-8.39.tar.gz
cd pcre-8.39
./configure
make && make install
```

### 安装zlib库
```shell
cd /opt
wget http://zlib.net/zlib-1.2.11.tar.gz
tar -zxvf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./configure
make && make install
```

### 安装nginx

```shell
cd /opt
wget -c https://nginx.org/download/nginx-1.10.1.tar.gz
tar -zxvf nginx-1.10.1.tar.gz
cd nginx-1.10.1
```


#### 配置编译

- 注意最后一行，配置fastdfs-nginx模块的路径（这个路径根据自己实际情况而定）

```shell
./configure \
--prefix=/usr/local/nginx \
--pid-path=/var/local/nginx/nginx.pid \
--lock-path=/var/lock/nginx/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi \
--add-module=/home/fastdfs-5.11/fastdfs-nginx-module/src

make && make install
```


#### 启动、运行

```shell
mkdir -p /var/temp/nginx/client
cp /usr/local/nginx/sbin/nginx /usr/local/bin/

#启运
nginx 

#重启
nginx -s reload

#查看运行
netstat -anp|grep 80

```

#### nginx编译选项

- make是用来编译的，它从Makefile中读取指令，然后编译。
- make install是用来安装的，它也从Makefile中读取指令，安装到指定的位置。
- configure命令是用来检测你的安装平台的目标特征的。它定义了系统的各个方面，包括nginx的被允许使用的连接处理的方法，比如它会检测你是不是有CC或GCC，并不是需要CC或GCC，它是个shell脚本，执行结束时，它会创建一个Makefile文件。nginx的configure命令支持以下参数：

```shell
--prefix=path    #定义一个目录，存放服务器上的文件 ，也就是nginx的安装目录。默认使用 /usr/local/nginx。
--sbin-path=path #设置nginx的可执行文件的路径，默认为  prefix/sbin/nginx.
--conf-path=path  #设置在nginx.conf配置文件的路径。nginx允许使用不同的配置文件启动，通过命令行中的-c选项。默认为prefix/conf/nginx.conf.
--pid-path=path  #设置nginx.pid文件，将存储的主进程的进程号。安装完成后，可以随时改变的文件名 ， 在nginx.conf配置文件中使用 PID指令。默认情况下，文件名 为prefix/logs/nginx.pid.
--error-log-path=path #设置主错误，警告，和诊断文件的名称。安装完成后，可以随时改变的文件名 ，在nginx.conf配置文件中 使用 的error_log指令。默认情况下，文件名 为prefix/logs/error.log.
--http-log-path=path  #设置主请求的HTTP服务器的日志文件的名称。安装完成后，可以随时改变的文件名 ，在nginx.conf配置文件中 使用 的access_log指令。默认情况下，文件名 为prefix/logs/access.log.
--user=name  #设置nginx工作进程的用户。安装完成后，可以随时更改的名称在nginx.conf配置文件中 使用的 user指令。默认的用户名是nobody。
--group=name  #设置nginx工作进程的用户组。安装完成后，可以随时更改的名称在nginx.conf配置文件中 使用的 user指令。默认的为非特权用户。
--with-select_module --without-select_module #启用或禁用构建一个模块来允许服务器使用select()方法。该模块将自动建立，如果平台不支持的kqueue，epoll，rtsig或/dev/poll。
--with-poll_module --without-poll_module #启用或禁用构建一个模块来允许服务器使用poll()方法。该模块将自动建立，如果平台不支持的kqueue，epoll，rtsig或/dev/poll。
--without-http_gzip_module #不编译压缩的HTTP服务器的响应模块。编译并运行此模块需要zlib库。
--without-http_rewrite_module  #不编译重写模块。编译并运行此模块需要PCRE库支持。
--without-http_proxy_module #不编译http_proxy模块。
--with-http_ssl_module #使用https协议模块。默认情况下，该模块没有被构建。建立并运行此模块的OpenSSL库是必需的。
--with-pcre=path  #设置PCRE库的源码路径。PCRE库的源码（版本4.4 - 8.30）需要从PCRE网站下载并解压。其余的工作是Nginx的./ configure和make来完成。正则表达式使用在location指令和 ngx_http_rewrite_module 模块中。
--with-pcre-jit #编译PCRE包含“just-in-time compilation”（1.1.12中， pcre_jit指令）。
--with-zlib=path #设置的zlib库的源码路径。要下载从 zlib（版本1.1.3 - 1.2.5）的并解压。其余的工作是Nginx的./ configure和make完成。ngx_http_gzip_module模块需要使用zlib 。
--with-cc-opt=parameters #设置额外的参数将被添加到CFLAGS变量。例如,当你在FreeBSD上使用PCRE库时需要使用:--with-cc-opt="-I /usr/local/include。.如需要需要增加 select()支持的文件数量:--with-cc-opt="-D FD_SETSIZE=2048".
--with-ld-opt=parameters #设置附加的参数，将用于在链接期间。例如，当在FreeBSD下使用该系统的PCRE库,应指定:--with-ld-opt="-L /usr/local/lib".
```