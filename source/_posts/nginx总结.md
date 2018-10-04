---
title: nginx总结
date: 2018-08-10 16:53:33
tags:
---
### 1.context:上下文
main context: events/http
server in http, location in server

### 2.nginx与apache对比
4G内存服务器+apache(prefork模式)一般只能处理3000个并发;
Nginx 0.8.15 + PHP 5.2.10 (FastCGI) 服务器在3万并发连接下，开启的10个Nginx进程消耗150M内存（15M \* 10=150M），开启的64个php-cgi进程消耗1280M内存（20M * 64=1280M），加上系统自身消耗的内存，总共消耗不到2GB内存。如果服务器内存较小，完全可以只开启25个php-cgi进程，这样php-cgi消耗的总内存数才500M
关键是nginx在3万的并发下,仍然速度飞快,所以说同等环境下,nginx是apache连接并发数的10倍.

### 3.nginx各种错误原因总结
502 Bad Gateway:
<1> php-fpm后台死掉或没有开启;

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
