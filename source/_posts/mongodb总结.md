---
title: mongodb总结
date: 2018-08-12 20:41:55
tags:
category: mongodb总结
layout: post
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
db.collectionName.find([query], [fields]).pretty(): 查看并美化所有数据
db.collectionName.find({'a':'b'}): 条件查询
db.collectionName.find({field:{$lt:ISODate('2018-08-05')}})[.count()]: 小于2018-08-05的数据[数量]
例: db.collectionName.find({"age": {$lt:20}}) //查询条件,相当于select * from collectionName where age < 20
db.collectionName.findOne([query], [fields], [options]): 查询一条,相当于select * from users limit 1

```

#### 5.3 DML:
```php
db.collectionName.insert([{'a': 'b', 'c': 'd'},{}]): 插入数据
db.collectionName.insertOne(obj, <optional>): 插入单条
db.collectionName.remove({query}, true): 条件删除[第一条数据]
db.collectionName.remove({}): 清空集合中所有的文档
db.collectionName.save(obj): 集合中不包含id或者id在表中不存在则插入,存在则更新(整体替换)
db.collectionName.update(query, obj, upsert bool): upsert为true时,条件存在则更新,不存在则插入obj[此处不包括query插入]
例子: 
db.users.find() //{"_id": 5, "username": "test10"}
db.users.update({"username":"test11"}, {"_id": 6, "age": 20, "gender": 1}, true)
db.users.find() //{"_id": 5, "username": "test10"}, {"_id": 6, "age": 20, "gender": 1}
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

### 6.修改器,操作符
```php
$set（更新字段）
$unset(删除字段)、
$inc(自增或自减)
$and、$or、$in、$nin、$nor、$exists（用于判断文档中是否包含某字段）
$push(向数组中尾部添加一个元素)
$pushAll(将数组中的所有值push)
$addToSet（向set集合中添加元素）
$pop(删除数组中的头部或尾部元素) 
$pull(删除数组中指定的值)
$size（根据数组的长度进行筛选）
$slice(返回数组中部分元素，如前几个、后几个、中间连续几个元素)
$elemMatch(用于匹配数组中的多个条件)
$where(自定义筛选条件，效率比较低，需要将bson转为js对象，不能使用索引，可以先使用普通查询过滤掉部分不满足条件
```

### 7.索引
```php
db.collectionName.getIndexes(): 获取索引
    [
        {
            "v" : 2,
            "key" : {
                "_id" : 1
            },
            "name" : "_id_",
            "ns" : "h5maker.users"
        }
    ]
db.collectionName.ensureIndex({field : 1}): //添加索引
db.collectionName.dropIndexes(): 删除collectionName上所有的索引
db.collectionName.dropIndexes({name: 1}): 删除collectionName上指定的索引

```
<center>未完待续.......</center>

### 8. node相关
```
//pm2按环境启动: 
pm2 start ecosystem.json --env production
//ecosystem.json文件

```






















