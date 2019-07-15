---
title: php常用总结
date: 2018-10-31 11:28:18
tags:
category: [php]
---

### 1.常用函数
```php
get_defined_vars  (PHP 4 >= 4.0.4, PHP 5) — 获取由所有已定义变量所组成的数组
get_defined_functions (PHP 4 >= 4.0.4, PHP 5) — 获取所有已经定义的函数
get_defined_constants (PHP 4 >= 4.1.0, PHP 5) —  获取关联数组的名字所有的常量和他们的价值
get_loaded_extensions (PHP 4, PHP 5) — 获取所有可用的模块
get_extension_funcs (PHP 4, PHP 5) — 获取指定模块的可用函数
get_declared_classes (PHP 4, PHP 5) —  获取由已定义类的名字所组成的数组
gettype(): 获取类型
is_type(): 判断是否是某种类型[is_int/is_integer,is_bool,is_string]
usleep($micro_time): 以指定的微妙数延迟程序的执行(1s=1000毫秒=1000000微妙)
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
/etc/init.d: 系统进程管理,command有start,stop,restart,reload等(service)
/etc/rc.local -> /etc/rc.d/rc.local: 开机启动

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
mac:
./configure --prefix=/usr/local/curl --with-ssl=/usr/local/Cellar/openssl/1.0.2q(没有openssl看下方下载方法)

wget http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.13.1.tar.gz
tar -zxvf libiconv-1.13.1.tar.gz
cd libiconv-1.13.1
./configure --prefix=/usr/local/libiconv
make
make install

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
--enable-pcntl \
--enable-sockets \
--with-iconv=/usr/local/libiconv \
--enable-fpm  #作用:产生php-fpm进程管理器以及配置文件

参数说明:
--with-config-file-path: 指定php.ini位置
--with-openssl: openssl的支持，https加密传输时用到的
--with-zlib: 打开zlib支持
--enable-bcmath: 允许使用高精度数学函数 
--enable-pcntl: pcntl扩展是PHP在Linux环境下进程控制的重要扩展，WorkerMan用到了其进程创建、信号控制、定时器、进程状态监控等特性。此扩展win平台不支持。

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
--with-openssl=/usr/local/Cellar/openssl/1.0.2q

make && make install
cp /usr/local/src/php-7.2.10/php.ini-development /usr/local/php/etc/php.ini
cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf
/usr/local/php/sbin/php-fpm && ps aux | grep php
service mysqld start #启动mysql
pkill -9 php-fpm #杀死php进程',

```

### 6. 自定义函数
#### 6.1 ip与int型互转
```php
//原理:无论是二进制移位,还是其他操作,原理都是以256(16^2)为单位进行处理
function ipToInt($ip) {
    $ipArr = explode('.', $ip);
    $ip = $ipArr[0] * 0x1000000
    + $ipArr[1] * 0x10000
        + $ipArr[2] *0x100
        + $ipArr[3];
    return $ip;
}

function ipToInt1($ip) {
    $ipArr = explode('.', $ip);
    $ip = $ipArr[0] * pow(16, 6)
        + $ipArr[1] * pow(16, 4)
        + $ipArr[2] * pow(16, 2)
        + $ipArr[3];
    return $ip;
}

function ipToInt2($ip) {
    $ipArr = explode('.', $ip);
    $ip = ($ipArr[0] << 24)
        + ($ipArr[1] << 16)
        + ($ipArr[2] << 8)
        + $ipArr[3];
    return $ip;
}

function intToIp($int) {
    $ipArr[0] = floor($int/pow(16, 6));
    $ipVint = $int - ($ipArr[0] * pow(16, 6));
    $ipArr[1] = ($ipVint & 0xFF0000) >> 16;
    $ipArr[2] = ($ipVint & 0xFF00) >> 8;
    $ipArr[3] = $ipVint & 0xFF;
    return implode('.', $ipArr);
}

function intToIp1($int) {
    $a = ($int >> 24) & 255;
    $b = ($int >> 16) & 255;
    $c = ($int >> 8) & 255;
    $d = $int & 255;
    return "$a.$b.$c.$d";
}
$ip = '192.168.200.109';
var_dump(ipToInt($ip));
var_dump(ipToInt1($ip));
var_dump(ipToInt2($ip));
var_dump(intToIp(ipToInt($ip)));
var_dump(intToIp1(ipToInt($ip)));
```

### 7. 手册总结
#### 7.10 heredoc与nowdoc的区别
```php
$heredoc = <<<EOD
my name is {$name}.
EOD;
//或者
$heredoc = <<<"EOD"
my name is {$name}.
EOD;

$nowdoc = <<<'EOT'
my name is {$name}.
EOT;

```
> 区别: heredoc会进行变量解析(相当于双引号),nowdoc不会进行变量解析(相当于单引号)
#### 7.11 php5.6和php7.0的区别:
 ```php
 1. php7.0删除<script language="php"></script>和aps(<% %>)标记
```

#### 7.12 phpredis中connect()和pconnect()的区别

pconnect建立后的连接并不随请求的结束而关闭,而是依赖于php-fpm进程,php-fpm进程不死,redis connect就一直
存在,直到空闲超时时自动断开.

#### 7.13 class总结
##### 7.13.1 final关键字
> 如果父类中的方法被声明为 final，则子类无法覆盖该方法。如果一个类被声明为 final，则不能被继承。
> 属性不能被定义为 final，只有类和方法才能被定义为 final。
##### 7.14.2 后期静态绑定
```php
class A {
    public static function foo() {
        static::who();
    }

    public static function who() {
        echo __CLASS__."\n";
    }
}

class B extends A {
    public static function test() {
        A::foo();
        parent::foo();
        self::foo();
    }

    public static function who() {
        echo __CLASS__."\n";
    }
}
class C extends B {
    public static function who() {
        echo __CLASS__."\n";
    }
}

C::test(); //输出: ACC
```

#### 7.14 substr与mb_substr的区别
```php
substr($str,$start, $len);
mb_substr($str, $start, $len, $charset); //$str可以是中文字符,以字为单位截取
mb_substr('中文', 0, 1, 'utf-8');
```

#### 7.15 curl模拟put/delete发送,并用php接收,jquery也可实现put/delete发送(更改method参数)
```php
curl -v -X PUT http://composer.com/test.php -d 'a=b&c=d' //-v:显示请求信息 -X:指定发送方式 -d:参数
$_SERVER['REQUEST_METHOD'] //接收请求方式,如put/delete/post
file_get_contents('php://input') //接收put参数
curl -v -X DELETE http://composer.com/test.php?a=b
$_SERVER['QUERY_STRING'] //接收delete数据

parse_str($str_param, $array) //解析参数字符串为数组,并赋值给$array

```




























































