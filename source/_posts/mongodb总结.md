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

### 4.相关命令
#### 4.1 DDL:
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
```

#### 4.2 DQL:
```php
db.collectionName.count(): 统计条数

```

#### 4.3 DML:
```php
db.collectionName.insert(document): 插入数据


```