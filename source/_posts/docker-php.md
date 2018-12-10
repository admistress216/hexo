---
title: docker.php
date: 2018-12-10 23:00:54
tags:
---
### 1. docker镜像原理
镜像的最底层是一个启动文件系统（bootfs）镜像，bootfs的上层镜像叫做根镜像，一般来说，根镜像是一个操作系统，例如Ubuntu、CentOS等.
用户的镜像必须构建于根镜像之上，在根镜像之上，用户可以构建出各种各样的其他镜像,一般来说,镜像是只读的,容器在镜像之上做读写操作.

### 2. docker命令总结
#### 2.1 docker基础命令
```php
docker run: 运行应用
docker stop: 停止应用
docker rm: 删除应用
docker inspect: 查看容器(用-f={{.State.Running}}筛选)
docker logs + container: 查看容器日志
docker cp 本地文件 container_name:/path/to/: 拷贝本地文件到容器
docker export container > /path/to/filename.tar: 备份容器
cat /path/to/filename.tar | docker import - imagename:tag :导入备份到镜像(docker run即可运行)

例子:
cat nginx1.tar  | docker import - nginx1:newer
docker run -itd -p 80:80 --name nginx nginx1 /bin/bash(记得加/bin/bash) 


```

#### 2.2 docker run总结
```php
-d: 后台运行容器，并返回容器ID；
-i: 以交互模式运行容器，通常与 -t 同时使用；
-p: 端口映射，格式为：主机(宿主)端口:容器端口
-t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
--name="nginx": 为容器指定一个名称；

例子:
docker run -itd -p 8080:80 --name nginx  nginx 运行image镜像
docker exec -it nginx /bin/bash //进入容器
```
#### 2.3 docker images总结
```php
RESPOSITORY: 
```