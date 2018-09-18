---
title: redis总结
date: 2018-08-18 11:56:29
tags:
---
### 1.what is redis?
Redis is an open source (BSD licensed), in-memory data structure store, used as a database, cache and message broker. It supports data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs and geospatial indexes with radius queries.
> redis和memcache的独特之处：
> <1>redis可以用来做存储(storage),而memcache只能用来做存储(cache)[这个特性源于redis的持久化功能]
> <2>memcache只能存储strings类型的数据,而redis可以存储strings,hashes,lists,sets,sorted set等数据

### 2.下载及安装
下载地址: [redis官网](http://redis.io)
安装: 
```php
//必要环境: 
yum -y install gcc tcl    //tcl用于make test
make
make install PREFIX=/usr/local/redis
cp  /usr/local/src/redis-4.0.11/redis.conf   /usr/local/redis/bin
修改配置:
daemonize yes
启动:
./redis-server redis.conf
```

> 注意:
> 如make时出现jemalloc/jemalloc.h错误,运行make MALLOC=libc指定分配器为libc即可
> 也可进行jemalloc安装:
> 在https://github.com/jemalloc/jemalloc/releases下载jemalloc
> 进行编译安装:./configure --prefix=/usr/local/jemalloc && make && make install
> 将jemalloc库写入系统:
> echo /usr/local/jemalloc/lib >> /etc/ld.so.conf && ldconfig
> ok

### 3.redis目录文件解释
> redis-benchmark: redis性能检测工具
> redis-check-aof: 检查aof日志工具
> reids-check-dump: 检查rbd日志工具

### 4.常用操作
#### 4.1 通用命令
```php
keys * : 查询所有(s*:查询以s开头的键,?匹配符也支持,sit[e|y]:查询site或sity)
flushall: 清除所有db中的key
flushdb: 清除当前db中的key
randomkey: 返回随机key
type key: 返回key的类型
exists key: 判断key是否存在
del key [key]: 删除key
rename key newkey: 重命名key
renamenx key newkey: 新key存在则不覆盖
move key db: 移动key到新db
select db: 切换db
ttl key: 查询key的生命周期,返回秒数,对于已存在的key[无限期]/不存在的key/过期的key都返回-1
expire key int: 设置key的生命周期(秒)
pexpire key int: 设置key的生命周期(毫秒)
pttl key: 查询key的生命周期(毫秒)
persist key: 设置key永久有效
```
#### 4.2 String命令
```php
set key value [ex 秒数]/[px 毫秒数] [nx]/[xx] //注意ex和px不要同时写,nx:key不存在时执行操作;xx:key存在时执行操作
mset key value [key value]: 一次性设置多个键值
mget key [key]: 一次性获取多个键值
setrange key offset value: 替换value
getrange key start end: 获取部分string  //下标左数从0开始,右数从-1开始
append key value: 在key的原value后添加value
getset key: 读取旧值并设置新值
incr/decr key: 加一/减一
incrby/decrby key int:一次加减int
incrby/decrbyfloat key float:一次加减float
setbit key offset value: 偏移量操作(单词大小写相差32)

```




















