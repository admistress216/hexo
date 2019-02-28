---
title: redis总结
date: 2018-08-18 11:56:29
tags:
categories:
- cache
---
### 1.what is redis?
Redis is an open source (BSD licensed), in-memory data structure store, used as a database, cache and message broker. It supports data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs and geospatial indexes with radius queries.
> redis和memcache的独特之处：
> <1>redis可以用来做存储(storage),而memcache只能用来做存储(cache)[这个特性源于redis的持久化功能]
> <2>memcache只能存储strings类型的数据,而redis可以存储strings,hashes,lists,sets,sorted set等数据

### 2.下载及安装
下载地址: [redis官网](http://redis.io)
        [git地址](https://github.com/MicrosoftArchive/redis/releases)
        [redis中文网](http://www.redis.cn/)
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
#### 4.3 List(链表)命令
```php
lpush/rpush key value [value]: 把value推进key头部/尾部
lrange key start end: 取元素,end为-1时取所有
lpop/rpop key: 弹出元素
lrem count value: 删除指定数量的值,count为负数时从尾部开始
ltrim key start end: 截取(得到截取后的key)
lindex key offset: 取出第offset位置的值
llen key: key的长度
linsert key after|before search value value1 [已废弃]
作用: 在key链表中寻找’search’,并在search值之前|之后,.插入value1
注: 一旦找到一个search后,命令就结束了,因此不会插入多个value1

rpoplpush source dest
作用: 把source的尾部拿出,放在dest的头部,
并返回 该单元值
例子:
rpush task a b c d
rpoplpush task job
lrange task 0 -1: a b c
lrange job 0 -1: d
业务逻辑:
1:Rpoplpush task bak
2:接收返回值,并做业务处理
3:如果成功,rpop bak 清除任务. 如不成功,下次从bak表里取任务

brpop ,blpop  key timeout
作用:等待弹出key的尾/头元素, 
Timeout为等待超时时间
如果timeout为0,则一直等待
场景: 长轮询Ajax,在线聊天时,能够用到


```
### 5. 事物
#### 5.1 redis与mysql事物对比

|      | mysql | redis |
| ---- | ---- | ----- |
| 开启 | start transaction | muitl |
| 语句 | 普通sql | 普通命令 |
| 失败 | rollback回滚 | discard取消 |
| 成功 | commit | exec |

> 注: rollback与discard 的区别
> 如果已经成功执行了2条语句, 第3条语句出错.
> Rollback后,前2条的语句影响消失.
> Discard只是结束本次事务,前2条语句造成的影响仍然还在

> 注:
> 在mutil后面的语句中, 语句出错可能有2种情况
> 1: 语法就有问题, 
> 这种,exec时,报错, 所有语句得不到执行
> 
> 2: 语法本身没错,但适用对象有问题. 比如 zadd 操作list对象
> Exec之后,会执行正确的语句,并跳过有不适当的语句.
> 
> (如果zadd操作list这种事怎么避免? 这一点,由程序员负责)

```php
思考: 
我正在买票
Ticket -1 , money -100
而票只有1张, 如果在我multi之后,和exec之前, 票被别人买了---即ticket变成0了.
我该如何观察这种情景,并不再提交

悲观的想法: 
世界充满危险,肯定有人和我抢, 给 ticket上锁, 只有我能操作. [悲观锁]

乐观的想法:
没有那么人和我抢,因此,我只需要注意,
--有没有人更改ticket的值就可以了 [乐观锁]

Redis的事务中,启用的是乐观锁,只负责监测key没有被改动.


具体的命令----  watch命令(两个terminal模拟)
例: 
redis 127.0.0.1:6379> watch ticket
OK
redis 127.0.0.1:6379> multi
OK
redis 127.0.0.1:6379> decr ticket
QUEUED
redis 127.0.0.1:6379> decrby money 100
QUEUED
redis 127.0.0.1:6379> exec
(nil)   // 返回nil,说明监视的ticket已经改变了,事务就取消了.
redis 127.0.0.1:6379> get ticket
"0"
redis 127.0.0.1:6379> get money
"200"


watch key1 key2  ... keyN
作用:监听key1 key2..keyN有没有变化,如果有变, 则事务取消

unwatch 
作用: 取消所有watch监听
```
### 6. 消息订阅

```php
使用办法:
订阅端: Subscribe 频道名称
发布端: publish 频道名称 发布内容

客户端例子:
redis 127.0.0.1:6379> subscribe news
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "news"
3) (integer) 1
1) "message"
2) "news"
3) "good good study"
1) "message"
2) "news"
3) "day day up"

服务端例子:
redis 127.0.0.1:6379> publish news 'good good study'
(integer) 1
redis 127.0.0.1:6379> publish news 'day day up'
(integer) 1

```
### 7.redis持久化

持久化方式:
- 1.快照rdb
- 2.日志aof(append only file)

#### 7.1 快照(rdb)
```php
save 900 1      // 900内,有1条写入,则产生快照 
save 300 1000   // 如果300秒内有1000次写入,则产生快照
save 60 10000  // 如果60秒内有10000次写入,则产生快照
(这3个选项都屏蔽,则rdb禁用)

stop-writes-on-bgsave-error yes  // 后台备份进程出错时,主进程停不停止写入?
rdbcompression yes    // 导出的rdb文件是否压缩
Rdbchecksum   yes //  导入rbd恢复时数据时,要不要检验rdb的完整性
dbfilename dump.rdb  //导出来的rdb文件名
dir ./  //rdb的放置路径

```

#### 7.2 日志(aof)
```php
Aof 的配置
appendonly yes # 是否打开 aof日志功能

appendfsync always   # 每1个命令,都立即同步到aof. 安全,速度慢
appendfsync everysec # 折衷方案,每秒写1次
appendfsync no      # 写入工作交给操作系统,由操作系统判断缓冲区大小,统一写入到aof. 同步频率低,速度快,
appendfilename "appendonly.aof" #文件存放位置,和rdb公用dir配置项

no-appendfsync-on-rewrite  yes: # 正在导出rdb快照的过程中,要不要停止同步aof
auto-aof-rewrite-percentage 100 #aof文件大小比起上次重写时的大小,增长率100%时,重写
auto-aof-rewrite-min-size 64mb #aof文件,至少超过64M时,重写

命令重写,cli下bgrewriteaof

注: 在dump rdb过程中,aof如果停止同步,会不会丢失?
答: 不会,所有的操作缓存在内存的队列里, dump完成后,统一操作.

注: aof重写是指什么?
答: aof重写是指把内存中的数据,逆化成命令,写入到.aof日志里.
以解决 aof日志过大的问题.

问: 如果rdb文件,和aof文件都存在,优先用谁来恢复数据?
答: aof,所以开启aof时数据库会变空(aof文件为空的话)

问: 2种是否可以同时用?
答: 可以,而且推荐这么做

问: 恢复时rdb和aof哪个恢复的快
答: rdb快,因为其是数据的内存映射,直接载入到内存,而aof是命令,需要逐条执行
```
#### 7.3 redis集群搭建
```php
Master配置:
1:关闭rdb快照(备份工作交给slave)
2:可以开启aof

slave配置:
1: 声明slaveof
2: 配置密码[如果master有密码]
3: [某1个]slave打开 rdb快照功能
4: 配置是否只读[slave-read-only]
5: 关闭aof
6. pid文件名称修改
```
> 集群缺陷:
> 每次salave断开后,(无论是主动断开,还是网络故障)
> 再连接master
> 
> 都要master全部dump出来rdb,再aof,即同步的过程都要重新执行1遍.
> 
> 所以要记住---多台slave不要一下都启动起来,否则master可能IO剧增





























































