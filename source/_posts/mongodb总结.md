---
title: mongodb总结
date: 2018-08-12 20:41:55
tags:
category: mongodb总结
layout: page
---

### 1.简介
mongodb是nosql中的一款产品，属于文档型数据库，存储的是文档（Bson,json的二进制化），内部引擎由javascript实现，所有和node.js搭配使用有天然的优势。

数据在存储时是以bson形式进行存储的，查询时数据转换为js对象，并可以通过js语法进行操作。
注意事项:
- MongoDB不支持事务和夺标连接将查询
- MongoDB中键值对是有序的,相同的键值对,不同顺序,属于不同的文档
- new Date(); 返回日期对象，属于日期类型，Date()函数返回日期字符串，在Shell中操作日期要使用日期类型， 日期类型是包含时区的

### 2.bin目录文件作用
> bsondump: 导出bson结构（可视化转化）
> mongodump: 整体导出bson数据
> mongorestore: 导入bson
> mongo: 客户端
> mongod: 服务端
> mongos: 查询路由，用于分片
> mongoexport: 导出json,csv,tsv格式
> mongoimport: 导入
> bson: 导出bson结构
> bsondump: 导出bson结构
> bsondump: 导出bson结构

### 3.mongod参数说明

> --bind_ip 绑定固定ip
> --bind_ip_all 绑定所有ip
> --dbpath 数据库存储目录
> --logpath log file to send write to instead of stdout - has to be a file, not directory
> --port 指定端口，default 27017
> --fork fork server process(后台demon运行)
> --smallfiles 小空间运行
> --directoryperdb each database will be stored in a separate directory(创建独立子目录)
> --logappend 追加日志
> --pidfilepath full path to pidfile,默认不创建
> --keyFile 集群标识（授权使用）
> --journal 启用日志
> --maxConns 最大连接并发数
> --notablescan 不允许表扫描
> /bin/mongo dbaddress --port 17720 --eval "db.shutdownServer()": 关闭服务

### 4.mongo参数说明
> --port/host 连接端口/主机
> --eval 解析javascript
> --username 用户名
> --password 密码
> --quit be less chatty
> --shell 运行完文件进入shell
> 

### 5.相关命令
#### 5.1 DDL:
```php
show dbs/databases: 显示数据库
db.dropDatabase(): 删除所在数据库
use dbname: 选库/隐式创建数据库
show tables/collections: 显示数据表
db.getName(): 显示所在数据库名称
db.version(): mongo版本
db.hostInfo(): 获取mongo所在服务器主机信息
db.createCollection(name, {size, capped, max}): 创建表
db.collectionName.drop(): 删除表
db.listCommands(): 列出数据库命令
print("hello"): 打印语句
exit 退出mongoClient

```

#### 5.2 DQL:
```php
db.collectionName.count(): 统计条数
db.collectionName.find(): 查看所有数据
db.collectionName.find({'a':'b'}): 条件查询
db.collectionName.find({field:{$lt:ISODate('2018-08-05')}})[.count()]: 小于2018-08-05的数据[数量]

```

#### 5.3 DML:
```php
db.collectionName.insert(document): 插入数据
db.collectionName.remove({'a':'b'}): 条件删除

```

#### 5.4 Help:
```php
db.help(): help on db methods
db.collectionName.help(): help on collection methods
sh.help(): sharding helpers
rs.help(): replica set helpers
help admin: administrative help
help connect: connecting to a db help
help keys: key shortcuts
help misc: misc things to know
help mr: mapreduce
```




























