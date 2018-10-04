---
title: mysql总结
date: 2018-10-05 00:34:56
tags:
---
### 1. 更改密码及导出
```php
'mysqladmin -uroot -proot password + 新密码', //更改密码
'mysql -uroot -p +数据库 < +sql路径',//导入sql文件
'mysqldump -uroot -p [-d] +数据库 > sql文件', //导出sql文件,-d:no data(仅结构)
```

### 2. linux下mysql安装
```php
groupadd mysql
useradd -g mysql -s /sbin/nologin -M mysql, //-s:定义shell,-M : 不建立根目录,-g:指定组
mkdir -p /data/mysql/data,
chown -R mysql:mysql /data/mysql,

yum -y install gcc gcc-c++ cmake ncurses-devel bison,
wget --no-check-certificate https://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-5.6.39.tar.gz, //下载

# cmake ./

# -DCMAKE_INSTALL_PREFIX=/usr/local/mysql          \    #安装路径

# -DMYSQL_DATADIR=/usr/local/mysql/data            \    #数据文件存放位置

# -DSYSCONFDIR=/etc                                \    #my.cnf路径

# -DWITH_MYISAM_STORAGE_ENGINE=1                   \    #支持MyIASM引擎

# -DWITH_INNOBASE_STORAGE_ENGINE=1                 \    #支持InnoDB引擎

# -DWITH_MEMORY_STORAGE_ENGINE=1                   \    #支持Memory引擎

# -DWITH_READLINE=1                                \    #快捷键功能(我没用过)

# -DMYSQL_UNIX_ADDR=/tmp/mysqld.sock               \    #连接数据库socket路径

# -DMYSQL_TCP_PORT=3306                            \    #端口

# -DENABLED_LOCAL_INFILE=1                         \    #允许从本地导入数据

# -DWITH_PARTITION_STORAGE_ENGINE=1                \    #安装支持数据库分区

# -DMYSQL_USER=mysql                                \   #mysqld运行用户

# -DEXTRA_CHARSETS=all                             \    #安装所有的字符集

# -DDEFAULT_CHARSET=utf8                           \    #默认字符

# -DDEFAULT_COLLATION=utf8_general_ci
# make && make install

cmake ./ \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql         \
-DMYSQL_DATADIR=/data/mysql/data                \
-DSYSCONFDIR=/etc                                \
-DWITH_MYISAM_STORAGE_ENGINE=1                   \
-DWITH_INNOBASE_STORAGE_ENGINE=1                 \
-DMYSQL_UNIX_ADDR=/data/mysql/mysqld.sock        \
-DMYSQL_TCP_PORT=3306                            \
-DENABLED_LOCAL_INFILE=1                         \
-DWITH_PARTITION_STORAGE_ENGINE=1                \
-DMYSQL_USER=mysql                               \
-DEXTRA_CHARSETS=all                             \
-DDEFAULT_CHARSET=utf8                           \
-DDEFAULT_COLLATION=utf8_general_ci

make&&make install,

chown -R mysql:mysql /usr/local/mysql/,
mv /usr/local/src/mysql-5.6.39/support-files/my-default.cnf /etc/my.cnf //选择覆盖

mv /usr/local/src/mysql-5.6.39/support-files/mysql.server /etc/init.d/mysqld //服务端启动放入服务中service mysqld start
chmod a+x /etc/init.d/mysqld,
chkconfig --level 345 mysqld on, //开机启动

echo "export PATH=/usr/local/mysql/bin/:$PATH" >> /etc/profile
source /etc/profile //设置环境变量,方便客户端启动

//初始化设置(user,配置文件,基础目录,数据目录)
/usr/local/mysql/scripts/mysql_install_db \
--user=mysql \
--defaults-file=/etc/my.cnf \
--basedir=/usr/local/mysql \
--datadir=/data/mysql/data

vim /etc/my.cnf
[mysqld]
user=mysql
basedir=/usr/local/mysql
default-storage-engine=Innodb
datadir=/data/mysql/data //放置于mysql模块
socket=/data/mysql/mysql.sock
[mysql]
socket=/data/mysql/mysql.sock //注意放在最下方

service mysqld reload
service mysqld start

/usr/bin/mysqladmin -u root password \'z\' //修改密码
mysql -u root -p  -P port //登录

说明:
参数说明:
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql //安装目录
-DMYSQL_DATADIR=/usr/local/mysql/data //数据库存放目录
-DWITH_MYISAM_STORAGE_ENGINE=1 //安装myisam存储引擎
-DWITH_INNOBASE_STORAGE_ENGINE=1 //安装innodb存储引擎
-DWITH_ARCHIVE_STORAGE_ENGINE=1 //安装archive存储引擎
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 //安装blackhole存储引擎
-DENABLED_LOCAL_INFILE=1 //允许从本地导入数据
-DDEFAULT_CHARSET=utf8 　　//使用utf8字符
-DDEFAULT_COLLATION=utf8_general_ci //校验字符
-DEXTRA_CHARSETS=all 　　//安装所有扩展字符集
-DMYSQL_TCP_PORT=3306 //MySQL监听端口
-DMYSQL_USER=mysql //MySQL用户名
其他参数:
-DWITH-EMBEDDED_SERVER=1 //编译成embedded MySQL library (libmysqld.a)
-DSYSCONFDIR=/etc //MySQL配辑文件
-DMYSQL_UNIX_ADDR=/tmp/mysqld.sock //Unix socket 文件路径
-DWITH_READLINE=1 //快捷键功能
-DWITH_SSL=yes //SSL
-DWITH_MEMORY_STORAGE_ENGINE=1 //安装memory存储引擎
-DWITH_FEDERATED_STORAGE_ENGINE=1 //安装frderated存储引擎
-DWITH_PARTITION_STORAGE_ENGINE=1 //安装数据库分区
-DINSTALL_PLUGINDIR=/usr/local/mysql/plugin //插件文件及配置路径
```

### 3. 设置权限
```php
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456aA' WITH GRANT OPTION;
flush privileges;
```
