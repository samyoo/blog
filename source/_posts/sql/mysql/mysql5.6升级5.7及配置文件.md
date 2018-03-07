
---
title: mysql5.7配置文件
date: 2018-03-07 14:29:44
tags: mysql
categories: 数据库
---

### 参数说明
#### max connections参数修改不生效，限制在214的问题。

   - 修改文件`/lib/systemd/system/mysql.service`，在最下面添加

```ini
  LimitNOFILE=4096
  LimitMEMLOCK=4096
```

  然后运行

```shell
systemctl daemon-reload
systemctl restart mysql.service
```

#### 去掉mysql5.7严格区分表名大小写问题

```ini
lower_case_table_names = 1
```

#### sql_mode 的修改

```ini
#ONLY_FULL_GRUOP_BY 可以兼容底版本的分组SQL
#NO_ZERO_IN_DATE 日期字段可以默认插入0值
#ERROR_FOR_DIVISION_BY_ZERO 可以把0做除数
sql_mode = ''
```

#### 修改utf编码    
网上很多资源都是在[mysqld]下添加`default-character-set=utf8`
如果这样改会导致5.7版本mysql无法打开所以要改为`character-set-server=utf8`

```ini
[client]
default-character-set=utf8
 
[mysqld]
character-set-server=utf8
```

#### 缓存的设置   
会话 的缓存大小，是针对每一个connection的，这个值也不会越大越好，默认大小是256kb，过大的配置会消耗更多的内存。


```ini
read_buffer_size        = 16M
read_rnd_buffer_size    = 16M
sort_buffer_size        = 16M
join_buffer_size        = 16M
```

#### innodb_buffer_pool_size 参数   
这个参数主要作用是缓存innodb表的索引，数据，插入数据时的缓冲,默认值：`128M`
专用mysql服务器设置的大小:**操作系统内存的70%-80%最佳。**
此外，这个参数是非动态的，要修改这个值，需要重启mysqld服务。所以设置的时候要非常谨慎。
并不是设置的越大越好。设置的过大，会导致system的swap空间被占用，导致操作系统变慢，从而减低sql查询的效率。

```ini
innodb_buffer_pool_size = 16G
```

#### innodb_flush_log_at_trx_commi 参数

    - 设置为1 代表`InnoDB`会在每次提交后刷新(fsync)事务日志到磁盘上,这提供了完整的 ACID 行为.
如果你愿意对事务安全折衷, 并且你正在运行一个小的事物, 你可以设置此值到0或者2来减少由事务日志引起的磁盘I/O   

    - 设置为0 代表日志只大约每秒写入日志文件并且日志文件刷新到磁盘.   
    - 设置为2 代表日志写入日志文件在每次提交后,但是日志文件只有大约每秒才会刷新到磁盘上.    

>说明：如果是游戏服务器，建议此值设置为2；如果是对数据安全要求极高的应用，建议设置为1；
设置为0性能最高，但如果发生故障，数据可能会有丢失的危险！
默认值1的意思是每一次事务提交或事务外的指令都需要把日志写入（flush）硬盘，这是很费时的。
特别是使用电池供电缓存（Battery backed up cache）时。
设成2对于很多运用，特别是从MyISAM表转过来的是可以的，它的意思是不写入硬盘而是写入系统缓存。
日志仍然会每秒flush到硬盘，所以你一般不会丢失超过1-2秒的更新。
设成0会更快一点，但安全方面比较差，即使MySQL挂了也可能会丢失事务的数据。而值2只会在整个操作系统挂了时才可能丢数据。）


```ini
innodb_flush_log_at_trx_commit = 2
```

#### innodb_thread_concurrency 参数

在`InnoDb`核心内的允许线程数量.最优值依赖于应用程序,硬件以及操作系统的调度方式.过高的值可能导致线程的互斥颠簸.

```ini
innodb_thread_concurrency = 12
```

#### query_cache_limit 与 query_cache_size
    - query_cache_size : 查询缓冲常被用来缓冲`SELECT`的结果并且在下一次同样查询的时候不再执行直接返回结果.打开查询缓冲可以极大的提高服务器速度, 如果你有大量的相同的查询并且很少修改表.
查看`Qcache_lowmem_prunes`状态变量来检查是否当前值对于你的负载来说是否足够高.

    - query_cache_limit : 只有小于此设定值的结果才会被缓冲此设置用来保护查询缓冲,防止一个极大的结果集将其他所有的查询结果都覆盖.

>***注意:*** 在你表经常变化的情况下或者如果你的查询原文每次都不同,
查询缓冲也许引起性能下降而不是性能提升.



```ini
#
# * Query Cache Configuration
#
query_cache_limit       = 2M
query_cache_size        = 32M
```


### 最后附上配置文件
```ini
#
# The MySQL database server configuration file.
#
# You can copy this to one of:
# - "/etc/mysql/my.cnf" to set global options,
# - "~/.my.cnf" to set user-specific options.
# 
# One can use all long options that the program supports.
# Run program with --help to get a list of available options and with
# --print-defaults to see which it would actually understand and use.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

# This will be passed to all mysql clients
# It has been reported that passwords should be enclosed with ticks/quotes
# escpecially if they contain "#" chars...
# Remember to edit /etc/mysql/debian.cnf when changing the socket location.

# Here is entries for some specific programs
# The following values assume you have at least 32M ram
[client]
default-character-set=utf8


[mysqld_safe]
socket		= /var/run/mysqld/mysqld.sock
nice		= 0

[mysqld]
#
# * Basic Settings
#
user		= mysql
pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
port		= 3306
basedir		= /usr
datadir		= /var/lib/mysql
tmpdir		= /tmp
lc-messages-dir	= /usr/share/mysql
skip-external-locking

open_files_limit 	= 8192
max_connections 	= 3000
max_connect_errors	= 500
tmp_table_size 		= 128M
max_allowed_packet	= 64M
interactive_timeout 	= 1800
wait_timeout 		= 1800
read_buffer_size 	= 16M
read_rnd_buffer_size	= 16M
sort_buffer_size 	= 16M
join_buffer_size 	= 16M
#sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION'
sql_mode                = ''

lower_case_table_names  = 1
group_concat_max_len    = 102400
character-set-server    = utf8

innodb_write_io_threads = 12
innodb_read_io_threads  = 12
innodb_buffer_pool_size = 12G
innodb_thread_concurrency = 12
innodb_flush_log_at_trx_commit = 2

#
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
#bind-address		= 127.0.0.1
#
# * Fine Tuning
#
key_buffer_size		= 64M
thread_stack		= 1M
thread_cache_size       = 32
# This replaces the startup script and checks MyISAM tables if needed
# the first time they are touched
myisam-recover-options  = BACKUP
#max_connections        = 100
#table_cache            = 64
#thread_concurrency     = 10
#
# * Query Cache Configuration
#
query_cache_limit	= 2M
query_cache_size        = 32M
#
# * Logging and Replication
#
# Both location gets rotated by the cronjob.
# Be aware that this log type is a performance killer.
# As of 5.1 you can enable the log at runtime!
#general_log_file        = /var/log/mysql/mysql.log
#general_log             = 1
#
# Error log - should be very few entries.
#
log_error = /var/log/mysql/error.log
#
# Here you can see queries with especially long duration
#log_slow_queries	= /var/log/mysql/mysql-slow.log
#long_query_time = 2
#log-queries-not-using-indexes
#
# The following can be used as easy to replay backup logs or for replication.
# note: if you are setting up a replication slave, see README.Debian about
#       other settings you may need to change.
#server-id		= 1
#log_bin			= /var/log/mysql/mysql-bin.log
expire_logs_days	= 10
max_binlog_size   = 300M
#binlog_do_db		= include_database_name
#binlog_ignore_db	= include_database_name
#
# * InnoDB
#
# InnoDB is enabled by default with a 10MB datafile in /var/lib/mysql/.
# Read the manual for more InnoDB related options. There are many!
#
# * Security Features
#
# Read the manual, too, if you want chroot!
# chroot = /var/lib/mysql/
#
# For generating SSL certificates I recommend the OpenSSL GUI "tinyca".
#
# ssl-ca=/etc/mysql/cacert.pem
# ssl-cert=/etc/mysql/server-cert.pem
# ssl-key=/etc/mysql/server-key.pem

```
