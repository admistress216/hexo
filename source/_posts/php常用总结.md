---
title: php常用总结
date: 2018-10-31 11:28:18
tags:
---

### 1.常用函数
```php
get_defined_vars  (PHP 4 >= 4.0.4, PHP 5) — 获取由所有已定义变量所组成的数组
get_defined_functions (PHP 4 >= 4.0.4, PHP 5) — 获取所有已经定义的函数
get_defined_constants (PHP 4 >= 4.1.0, PHP 5) —  获取关联数组的名字所有的常量和他们的价值
get_loaded_extensions (PHP 4, PHP 5) — 获取所有可用的模块
get_extension_funcs (PHP 4, PHP 5) — 获取指定模块的可用函数
get_declared_classes (PHP 4, PHP 5) —  获取由已定义类的名字所组成的数组
```

### 2.xdebug常用配置
```php
zend_extension="xdebug.so"
; 重写var_dump()函数,默认为2
xdebug.overload_var_dump=2
; var_dump()显示层数
xdebug.var_display_max_depth=4
; 启用追踪
xdebug.auto_trace="on"
xdebug.trace_output_dir="/Users/cctv/Documents/pro1/test/tmp"
xdebug.trace_output_name="trace"
xdebug.trace_options=1
; 启动性能分析
xdebug.profiler_enable=1
xdebug.profiler_output_dir="/Users/cctv/Documents/pro1/test/tmp"
```
> xdebug与phpstorm 联合使用

```php
; 开启远程调试
xdebug.remote_enable=1
xdebug.remote_handler="dbgp"
xdebug.remote_host="localhost"
xdebug.remote_port=9000
xdebug.remote_log="/Users/cctv/Documents/pro1/test/tmp/xdebug.log"
xdebug.idekey=PHPSTORM
;当此设置设置为1时，即使GET / POST / COOKIE变量不存在，Xdebug也将始终尝试启动远程调试会话并尝试连接到客户端。
xdebug.remote_autostart=1

;配置phpstorm->Languages & Frameworks
1. 配置php->php language leavel
2. 配置php->debug中的debug_port为remote_port
3. 配置php->debug->debug_proxy中idekey->idekey;host->虚拟主机文件路径,如:test.com/index.php;port->remote_port
4. 配置php->servers:host,port,xdebug(本地虚拟主机域名)
4. 配置editconfigure:server(test.com),start_url(/)
```

### 3. /etc/init.d和/etc/rc.local
/etc/init.d: 系统进程管理,command有start,stop,restart,reload等
/etc/rc.local: 开机启动

### 4. php-fpm相关
#### 4.1 /etc/init.d/php-fpm脚本内容

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
#### 4.2 配置
- php-fpm-www1.conf
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
- php-fpm-www2.conf
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

- rc.local
```php
/etc/init.d/php-fpm start #追加
```

### 5. php环境编译安装
```php
yum -y install mysql mysql-devel mysql-server //装mysql
yum -y install gd gd-devel freetype libxml2-devel
wget https://curl.haxx.se/download/curl-7.60.0.tar.gz --no-check-certificate
tar zxvf curl-7.60.0.tar.gz && cd curl-7.60.0
./configure --prefix=/usr/local/curl && make && make install

编译php(连接mysql,gd,ttf并以fpm/fastcgi方式运行)[nginx方式]:
./configure --prefix=/usr/local/php \
--with-config-file-path=/usr/local/php/etc \
--enable-mysqlnd \
--enable-pdo \
--enable-mbstring \
--with-pdo-mysql \
--with-mysqli \
--with-gd \
--with-openssl \
--with-zlib \
--enable-zip \
--with-freetype-dir=/usr/include/freetype2/freetype \
--with-curl=/usr/local/curl \
--enable-gd-jis-conv \
--enable-fpm  #作用:产生php-fpm进程管理器以及配置文件

参数说明:
--with-config-file-path: 指定php.ini位置
--with-openssl: openssl的支持，https加密传输时用到的
--with-zlib: 打开zlib支持
--enable-bcmath: 允许使用高精度数学函数 

[apache方式]:
./configure --prefix=/usr/local/fastphp --with-mysql=mysqlnd \
--enable-mysqlnd \
--enable-pdo \
--with-pdo-mysql \
--with-mysqli \
--with-gd \
--enable-gd-native-ttf \
--enable-gd-jis-conv \
--with-apxs2=/usr/local/httpd/bin/apxs //作用:将php作为apache子模块

configure: error: Cannot find OpenSSL's <evp.h>报错解决:
windows:
yum -y install openssl-devel
mac:
brew upgrade openssl
--with-openssl=/usr/local/Cellar/openssl/1.0.2p

make && make install
cp /usr/local/src/php-7.2.10/php.ini-development /usr/local/php/etc/php.ini
cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
/usr/local/php/sbin/php-fpm && ps aux | grep php
service mysqld start #启动mysql
pkill -9 php-fpm #杀死php进程',

```


































































