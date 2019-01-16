---
layout: 源码安装memcached和php
title: memcache扩展
date: 2018-10-07 15:10:34
tags:
categories:
- cache
---
### 1. 安装memcached依赖libevent和下载源码包
```php
yum install libevent libevent-devel
```
[官方下载](http://www.memcached.org/)

### 2. 安装memcached
#### 2.1 创建memcached用户
#useradd -d /var/run/memcached/ -s /sbin/nologin -c "Memcached daemon" memcached

#### 2.2 解压源码包并编译安装
#tar zxvf memcached-1.4.24.tar.gz
#cd memcached-1.4.24
#./configure --prefix=/usr/local/memcached
#make && make install

#### 2.3 添加service脚本
#vi /etc/rc.d/init.d/memcached
内容如下:
```php
#! /bin/sh
#
# chkconfig: - 55 45
# description: The memcached daemon is a network memory cache service.
# processname: memcached
# config: /etc/sysconfig/memcached
# pidfile: /var/run/memcached/memcached.pid

# Standard LSB functions
#. /lib/lsb/init-functions

# Source function library.
. /etc/init.d/functions

IP=127.0.0.1
PORT=11211
USER=memcached
MAXCONN=1024
CACHESIZE=64
OPTIONS=""

# Check that networking is up.
. /etc/sysconfig/network

if [ "$NETWORKING" = "no" ]
then

exit 0
fi

RETVAL=0
prog="memcached"
pidfile=${PIDFILE-/var/run/memcached/memcached.pid}
lockfile=${LOCKFILE-/var/lock/subsys/memcached}

start () {

echo -n $"Starting $prog: "
# Ensure that /var/run/memcached has proper permissions
if [ "`stat -c %U /var/run/memcached`" != "$USER" ]; then
    chown $USER /var/run/memcached
fi

daemon --pidfile ${pidfile}  /usr/local/memcached/bin/memcached -d -l $IP  -p $PORT -u $USER  -m $CACHESIZE -c $MAXCONN -P ${pidfile} $OPTIONS
RETVAL=$?
echo
[ $RETVAL -eq 0 ] && touch ${lockfile}
}
stop () {

echo -n $"Stopping $prog: "
killproc -p ${pidfile} /usr/local/memcached/bin
RETVAL=$?
echo
if [ $RETVAL -eq 0 ] ; then
    rm -f ${lockfile} ${pidfile}
fi
}

restart () {

    stop
    start
}

# See how we were called.
case "$1" in
start)

start
;;
stop)

stop
;;
status)

status -p ${pidfile} memcached
RETVAL=$?
;;
restart|reload|force-reload)

restart
;;
condrestart|try-restart)

[ -f ${lockfile} ] && restart || :
;;
*)

echo $"Usage: $0 {start|stop|status|restart|reload|force- reload|condrestart|try-restart}"
RETVAL=2
    ;;
esac

exit $RETVAL
```

#### 2.4启动并测试
chmod a+x /etc/init.d/memcached
service memcached start
telnet 127.0.0.1 11211
stats
退出stats和telnet ctrl+] quit

### 3. 安装php memcache

#### 3.1 安装依赖
```php
wget https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gz
tar zxf libmemcached-1.0.18.tar.gz
cd libmemcached-1.0.18
./configure --prefix=/usr/local/libmemcached --with-memcached
make && make install
```

#### 3.2 下载安装php memcache扩展
> 注意:php7以外可以在pecl扩展库下载,下面介绍php7的情况
```php
git clone https://github.com/php-memcached-dev/php-memcached.git
cd php-memcached
git checkout php7
/usr/local/php/bin/phpize
./configure --enable-memcached --with-php-config=/usr/local/php/bin/php-config --with-libmemcached-dir=/usr/local/libmemcached  --disable-memcached-sasl
make
make install

//php.ini中添加
[Memcached]
extension=memcached.so
//重启php-fpm
service php-fpm restart
```
