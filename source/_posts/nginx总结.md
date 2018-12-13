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
Nginx 0.8.15 + PHP 5.2.10 (FastCGI) 服务器在3万并发连接下(单台物理机可支持30000~50000个并发)，开启的10个Nginx进程消耗150M内存（15M \* 10=150M），开启的64个php-cgi进程消耗1280M内存（20M * 64=1280M），加上系统自身消耗的内存，总共消耗不到2GB内存。如果服务器内存较小，完全可以只开启25个php-cgi进程，这样php-cgi消耗的总内存数才500M
关键是nginx在3万的并发下,仍然速度飞快,所以说同等环境下,nginx是apache连接并发数的10倍.

#### 1.3 下载安装与启动
- gcc安装: 用于源码编译
- pcre安装: PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库。
- zlib安装: zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库
- openssl安装: OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库

```php
yum -y install gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel //缺少pcre(用于正则)和zlib(压缩算法)的情况
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
client_body_buffer_size 128k;  //缓存大小
```

#### 6.1 会话持久性
- 方式一: 用ip_hash实现(基于ip实现)
```php

upstream backend {
    ip_hash;
    server 127.0.0.1:9000;
    server 127.0.0.1:9001;
}

```
> 存在问题:
> 当后端服务器宕机后，session会丢失；
> 来自同一局域网的客户端会被转发到同一个后端服务器，可能导致负载失衡；
> 不适用于CDN网络，不适用于前段还有代理的情况。

- 方式二: nginx-sticky-module
```php
./configure --prefix=/usr/local/nginx --add-module=../nginx-sticky-1.2.5

//报错
/root/nginx-sticky-1.2.5//ngx_http_sticky_misc.c: 在函数‘ngx_http_sticky_misc_sha1’中:
/root/nginx-sticky-1.2.5//ngx_http_sticky_misc.c:176:15: 错误：‘SHA_DIGEST_LENGTH’未声明(在此函数内第一次使用)
   u_char hash[SHA_DIGEST_LENGTH];
               ^
/root/nginx-sticky-1.2.5//ngx_http_sticky_misc.c:176:15: 附注：每个未声明的标识符在其出现的函数内只报告一次
/root/nginx-sticky-1.2.5//ngx_http_sticky_misc.c:176:10: 错误：未使用的变量‘hash’ [-Werror=unused-variable]
   u_char hash[SHA_DIGEST_LENGTH];
          ^
/root/nginx-sticky-1.2.5//ngx_http_sticky_misc.c: 在函数‘ngx_http_sticky_misc_hmac_sha1’中:
/root/nginx-sticky-1.2.5//ngx_http_sticky_misc.c:242:15: 错误：‘SHA_DIGEST_LENGTH’未声明(在此函数内第一次使用)
   u_char hash[SHA_DIGEST_LENGTH];

//解决(修改ngx_http_sticky_misc.c文件，新增#include <openssl/sha.h>和#include <openssl/md5.h>模块):
sed -i '12a #include <openssl/sha.h>' /root/nginx-sticky-1.2.5/ngx_http_sticky_misc.c
sed -i '12a #include <openssl/sha.h>' /root/nginx-sticky-1.2.5/ngx_http_sticky_misc.c
```

```php
upstream {
  sticky;
  server 127.0.0.1:9000;
  server 127.0.0.1:9001;
  server 127.0.0.1:9002;
}

参数说明:
sticky [name=route] [domain=.foo.bar] [path=/] [expires=1h] 
    [hash=index|md5|sha1] [no_fallback] [secure] [httponly];

[name=route]　　　　　　　设置用来记录会话的cookie名称
[domain=.foo.bar]　　　　设置cookie作用的域名
[path=/]　　　　　　　　  设置cookie作用的URL路径，默认根目录
[expires=1h] 　　　　　　 设置cookie的生存期，默认不设置，浏览器关闭即失效，需要是大于1秒的值
[hash=index|md5|sha1]   设置cookie中服务器的标识是用明文还是使用md5值，默认使用md5
[no_fallback]　　　　　　 设置该项，当sticky的后端机器挂了以后，nginx返回502 (Bad Gateway or Proxy Error) ，而不转发到其他服务器，不建议设置
[secure]　　　　　　　　  设置启用安全的cookie，需要HTTPS支持
[httponly]　　　　　　　  允许cookie不通过JS泄漏，没用过
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

#### 7.2 nginx各变量总结
> [POST] 请求http://192.168.99.131/sub?name=liyi&age=18:
> $query_string: get地址参数(name=liyi&age=18)
> $request_uri: uri(/sub?name=liyi&age=18)
> $echo_request_method: 请求的方式(post)
> $document_root: nginx配置的root(目录地址)
> $fastcgi_script_name: 请求的uri,无参数(/sub);
> $uri: 请求的uri,无参数(/sub)
> $nginx_version: nginx版本
> $remote_addr/$remote_port: 请求端ip/port(192.168.99.1/51796)[使用虚拟机]
> $server_addr/$server_port: 服务端ip/port(192.168.99.131/80)
> $content_length: body字节数
> $content_type: 获取enctype(对表单数据的编码格式,浏览器默认application/x-www-form-urlencoded,上传文件时multipart/form-data,纯文本文件text/plain)
> [官方系统嵌入式变量地址](http://nginx.org/en/docs/http/ngx_http_core_module.html#variables)

### 8. 第三方模块
#### 8.1 各模块地址
- ngx_devel_kit https://github.com/simpl/ngx_devel_kit
- set-misc-nginx-module https://github.com/agentzh/set-misc-nginx-module
- memc-nginx-module https://github.com/agentzh/memc-nginx-module
- echo-nginx-module https://github.com/agentzh/echo-nginx-module
- lua-nginx-module https://github.com/chaoslawful/lua-nginx-module
- srcache-nginx-module https://github.com/agentzh/srcache-nginx-module
- drizzle-nginx-module https://github.com/chaoslawful/drizzle-nginx-module
- rds-json-nginx-module https://github.com/agentzh/rds-json-nginx-module
- luajit下载地址 https://github.com/openresty/luajit2/releases
- lua-resty-redis https://github.com/openresty/lua-resty-redis


> luajit安装方式:
> make install PREFIX=/usr/local/luajit
> export LUAJIT_LIB=/usr/local/luajit/lib
> export LUAJIT_INC=/usr/local/luajit/include/luajit-2.1

### 8.2 编译安装
```php
./configure --prefix=/usr/local/nginx \
         --with-ld-opt="-Wl,-rpath,/usr/local/luajit/lib" \
         --add-module=/usr/local/src/ngx_devel_kit \
         --add-module=/usr/local/src/lua-nginx-module \
         --add-module=/usr/local/src/echo-nginx-module \
         --add-module=/usr/local/src/nginx-sticky-module-ng \
         --add-module=/usr/local/src/redis2-nginx-module \
         --add-module=/usr/local/src/set-misc-nginx-module
```

#### 8.3 ngx_lua总结
```php
ngx.req.read_body()  //开始读取body数据
ngx.req.get_body_data() //获取body数据
ngx.req.get_body_file() //获取body文件

```

#### 8.4 cjson安装
```php

//Makefile中:
##### Build defaults #####
LUA_VERSION =       5.1
TARGET =            cjson.so
PREFIX =            /usr/local/luajit
#CFLAGS =            -g -Wall -pedantic -fno-inline
CFLAGS =            -O3 -Wall -pedantic -DNDEBUG
CJSON_CFLAGS =      -fpic
CJSON_LDFLAGS =     -shared
LUA_INCLUDE_DIR =   $(PREFIX)/include/luajit-2.1
#LUA_CMODULE_DIR =   $(PREFIX)/lib/lua/$(LUA_VERSION)
LUA_CMODULE_DIR =   $(PREFIX)/lib
#LUA_MODULE_DIR =    $(PREFIX)/share/lua/$(LUA_VERSION)
LUA_MODULE_DIR =    $(PREFIX)/share/lua/luajit-2.1.0-beta3
LUA_BIN_DIR =       $(PREFIX)/bin

make && make install
//如果报静态错误,修改lua_cjson.c,去掉static修饰

//nginx引入cpath
lua_package_cpath '/usr/local/luajit/lib/?.so;;';
```