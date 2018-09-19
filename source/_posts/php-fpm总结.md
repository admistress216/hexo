---
title: php-fpm总结
date: 2018-09-08 00:01:18
tags:
---

### 1. /etc/init.d和/etc/rc.local
/etc/init.d: 系统进程管理,command有start,stop,restart,reload等
/etc/rc.local: 开机启动

### 2. /etc/init.d/php-fpm脚本内容

```php
#!/bin/bash
#
# Startup script for the PHP-FPM server.
#
# chkconfig: 345 85 15
# description: PHP is an HTML-embedded scripting language
# processname: php-fpm
# config: /usr/local/php/etc/php.ini

# Source function library.
. /etc/rc.d/init.d/functions

PHP_PATH=/usr/local
DESC="php-fpm daemon"
NAME=php-fpm
NAME1=php-fpm-www1
NAME2=php-fpm-www2
# php-fpm路径
DAEMON=$PHP_PATH/php/sbin/$NAME
# 配置文件路径
CONFIGFILE1=$PHP_PATH/php/etc/$NAME1.conf
CONFIGFILE2=$PHP_PATH/php/etc/$NAME2.conf
# PID文件路径(在php-fpm.conf设置)
PIDFILE1=$PHP_PATH/php/var/run/$NAME1.pid
PIDFILE2=$PHP_PATH/php/var/run/$NAME2.pid
SCRIPTNAME=/etc/init.d/$NAME

# Gracefully exit if the package has been removed.
test -x $DAEMON || exit 0

rh_start() {
  $DAEMON -y $CONFIGFILE1 || echo -e " already running"
  $DAEMON -y $CONFIGFILE2 || echo -e " already running"
}

rh_stop() {
  kill -QUIT `cat $PIDFILE1` || echo -e " not running"
  kill -QUIT `cat $PIDFILE2` || echo -e " not running"
}

rh_reload() {
  kill -HUP `cat $PIDFILE1` || echo -e " can't reload"
  kill -HUP `cat $PIDFILE2` || echo -e " can't reload"
}

case "$1" in
  start)
        echo -e "Starting $DESC: $NAME1"
        echo -e "Starting $DESC: $NAME2"
        rh_start
        echo "."
        ;;
  stop)
        echo -e "Stopping $DESC: $NAME1"
        echo -e "Stopping $DESC: $NAME2"
        rh_stop
        echo "."
        ;;
  reload)
        echo -e "Reloading $DESC configuration..."
        rh_reload
        echo "reloaded."
  ;;
  restart)
        echo -e "Restarting $DESC: $NAME1"
        echo -e "Restarting $DESC: $NAME2"
        rh_stop
        sleep 1
        rh_start
        echo "."
        ;;
  *)
         echo "Usage: $SCRIPTNAME {start|stop|restart|reload}" >&2
         exit 3
        ;;
esac
```
### 3. 配置
#### 3.1 php-fpm-www1.conf
```php
[global]
pid = /usr/local/php/var/run/php-fpm-www1.pid
error_log =/usr/local/php/var/log/php-fpm-www1.log

; Possible Values: alert, error, warning, notice, debug
log_level = warning

emergency_restart_threshold = 10
emergency_restart_interval = 60s

process_control_timeout = 5s
process.max = 2048

daemonize = yes
rlimit_files = 65536
rlimit_core = 0
events.mechanism = epoll
; Default value: 10
;systemd_interval = 10

[www1]
user = www
group = www
;fpm监听端口/套接字,/dev/shm是个tmpfs,速度比磁盘快的多
listen = /dev/shm/php5-fpm-www1.sock
listen.backlog = 65535
;运行用户
listen.owner = www
;运行组
listen.group = www
;运行用户和组所使用的权限
listen.mode = 0660

;子进程数固定
pm = static
;进程数量(static)/最大数量(dynamic)
pm.max_children = 200

;多久后空闲结束进程,仅当pm设置为ondemand时有效
pm.process_idle_timeout = 30s
;子进程重生之前服务的请求数量
pm.max_requests = 20000

; The access log file
; Default: not set
access.log = /usr/local/php/var/log/$pool.access.log

slowlog = /usr/local/php/var/log/$pool.log.slow
request_slowlog_timeout = 5s
request_terminate_timeout = 3m
rlimit_files = 65000

;使用 php_admin_value 或者 php_admin_flag 定义的值，不能被 PHP 代码中的 ini_set() 覆盖
php_admin_value[error_log] = /usr/local/php/var/log/fpm-php.www1.log
;重定向运行过程中的 stdout 和 stderr 到主要的错误日志文件中。如果没有设置，stdout 和 stderr 将会根据 FastCGI 的规则被重定向到 /dev/null。默认值：无。
catch_workers_output = yes
```
#### 3.2 php-fpm-www2.conf
```php
[global]
pid = /usr/local/php/var/run/php-fpm-www2.pid
error_log =/usr/local/php/var/log/php-fpm-www2.log

; Possible Values: alert, error, warning, notice, debug
log_level = warning

emergency_restart_threshold = 10
emergency_restart_interval = 60s

process_control_timeout = 10s
process.max = 2048

daemonize = yes
rlimit_files = 65536
rlimit_core = 0
events.mechanism = epoll
; Default value: 10
;systemd_interval = 10

[www2]
user = www
group = www
listen = /dev/shm/php5-fpm-www2.sock
listen.backlog = 65535
listen.owner = www
listen.group = www
listen.mode = 0660

pm = static
pm.max_children = 200

pm.process_idle_timeout = 30s
pm.max_requests = 20000

; The access log file
; Default: not set
access.log = /usr/local/php/var/log/$pool.access.log

slowlog = /usr/local/php/var/log/$pool.log.slow
request_slowlog_timeout = 5s
request_terminate_timeout = 3m
rlimit_files = 65000

php_admin_value[error_log] = /usr/local/php/var/log/fpm-php.www2.log
catch_workers_output = yes
```

#### 3.3 rc.local
```php
/etc/init.d/php-fpm start #追加
```

### 4.配置解析
