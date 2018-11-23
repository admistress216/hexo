---
title: nginx总结
date: 2018-08-10 16:53:33
tags:
---
### 1. 基础知识
#### 1.1 nginx配置context:上下文
main context: events/http
server in http, location in server

#### 1.2 nginx与apache对比
4G内存服务器+apache(prefork模式)一般只能处理3000个并发;
Nginx 0.8.15 + PHP 5.2.10 (FastCGI) 服务器在3万并发连接下，开启的10个Nginx进程消耗150M内存（15M \* 10=150M），开启的64个php-cgi进程消耗1280M内存（20M * 64=1280M），加上系统自身消耗的内存，总共消耗不到2GB内存。如果服务器内存较小，完全可以只开启25个php-cgi进程，这样php-cgi消耗的总内存数才500M
关键是nginx在3万的并发下,仍然速度飞快,所以说同等环境下,nginx是apache连接并发数的10倍.

#### 1.3 下载安装与启动
```php
yum -y install pcre pcre-devel zlib-devel //缺少pcre(用于正则)和zlib(压缩算法)的情况
./configure --prefix=/usr/local/nginx
make && make install
./nginx //启动
netstat -antp //查看端口和占用程序
```

#### 1.4 信号量操作
```php
kill+signal+pid //格式
kill -INT/TERM +PID //shutdown quickly
kill -QUIT +PID //Graceful shutdown 优雅的关闭进程,等请求都结束后再关闭
kill -HUP +PID //new processes,new config,and graceful shutdown old processes.
kill -USR1+PID //用于日志分割
kill -HUP `cat logs/nginx.pid` //不用输入pid的方式重启
./nginx -h //查看命令方式操作
```

> 虚拟主机可以根据ip,域名,port建立

### 2. nginx各种错误原因总结
#### 2.1 502 Bad Gateway:
<1> php-fpm后台死掉或没有开启;
<2> 超过php-fpm的响应时间: request_terminate_timeout(用于php.ini超时时php-fpm停止此子进程)

#### 2.2 504 Timeout:
<1> 超过nginx中fastcgi_read_timeout的设置时间

### 4. /etc/init.d/nginx脚本
```php
#! /bin/bash
# chkconfig: - 85 15
PATH=/usr/local/openresty/nginx
DESC="nginx daemon"
NAME=nginx
DAEMON=$PATH/sbin/$NAME
CONFIGFILE=$PATH/conf/$NAME.conf
PIDFILE=$PATH/logs/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME
set -e
[ -x "$DAEMON" ] || exit 0
do_start() {
$DAEMON -c $CONFIGFILE || echo -n "nginx already running"
}
do_stop() {
$DAEMON -s stop || echo -n "nginx not running"
}
do_reload() {
$DAEMON -s reload || echo -n "nginx can't reload"
}
case "$1" in
start)
echo -n "Starting $DESC: $NAME"
do_start
echo "."
;;
stop)
echo -n "Stopping $DESC: $NAME"
do_stop
echo "."
;;
reload|graceful)
echo -n "Reloading $DESC configuration..."
do_reload
echo "."
;;
restart)
echo -n "Restarting $DESC: $NAME"
do_stop
do_start
echo "."
;;
*)
echo "Usage: $SCRIPTNAME {start|stop|reload|restart}" >&2
exit 3
;;
esac
exit 0
```

### 5.nginx配置
#### 5.1 nginx.conf
```php
user  www www;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    upstream unix_tmp_php_cgi_sock {  #nginx负载均衡
        server unix:/dev/shm/php5-fpm-www1.sock;
        server unix:/dev/shm/php5-fpm-www2.sock;
    }
    include vhost/*.conf;

        }
```

- 配置分析
> worker_processes, //工作进程个数配置(子进程),可设置,太大无意义,争夺cpu,一般设置为cpu数*核数(多了争夺cpu)
> events, //一般是配置nginx连接特性,如一个worker能同时允许多少连接
> events.worker_connections, //一个子进程允许的最大连接数
> http, //主要是用来配置服务器(http大段内又包括各Server小段)
> http.server, //用来配置虚拟主机

- log参数解析
> $remote_addr: 远程ip',
> $remote_user[$time_local]: 访问时间',
> $request: post/get',
> $status: 404,200等状态码',
> $body_byte_sent: body体字节数',
> $http_referer: 访问来源',
> $http_user_agent: 用户代理(浏览器类型等)信息',
> $http_x_forwarded_for: 中间代理x'

#### 5.2 vhost文件夹下server.conf
```php
server {
        listen       80;
        server_name  localhost;

        location / {
            root   /project/html;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
        server {
                listen 8080;
                server_name localhost;

                location / {
                        root /project/php;
                        index index.html index.php;
                }
                error_page 500 503 504 /50x.html;
                location /50x.html {
                        root /project/php;
                }
                location ~ \.php {
                        root /project/php;
                        fastcgi_pass unix_tmp_php_cgi_sock; #通过socket连接(存在多个php-fpm线程池则填写负载名称)
                        fastcgi_index index.php;
                        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                        include fastcgi_params;
                }
        }
```

### 6. 配置解读
```php
#设置允许发布内容为8M(允许上传大小)
client_max_body_size 8M;
client_body_buffer_size 128k;
```

### 7. 其他功能
#### 7.1 定时任务与日志切割
```php
'date命令' => 'date -d yesterday +%y%m%d', //y:年份的后两位
'crontab命令' => '分 时 日 月 周',
'shell script' => '
    #!/bin/bash
    LOGPATH=/usr/local/nginx/logs/z.com.access.log
    BASEPATH=/data/$(date -d yesterday +%Y%m)
    
    mkdir -p $BASEPATH
    bak=$BASEPATH/$(date -d yesterday +%d%H%M).zcom.access.log
    mv LOGPATH $bak
    touch $LOGPATH //继续往$bak里写,与linux文件机制inode有关
    
    kill -USR1 $(cat /usr/local/nginx/logs/nginx.pid) //重写新的日志文件
',
'crontab' => '*/1 * * * * sh /data/runlog.sh', //crontab命令
```
