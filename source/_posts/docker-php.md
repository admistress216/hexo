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
docker info: 查看docker的详细信息
docker run: 运行应用(本地没有则自动下载)
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

docker images: 查看镜像
docker rmi image-name: 删除镜像
docker search image: 搜索镜像
docker pull image-name: 下载镜像
docker commit containerid respository-name: 从容器中创建本地镜像(仓库)
    1. 参数-m是对创建的该镜像的一个简单描述。
    2. --author表示该镜像的作者。
    3. ce1fe32739402表示创建镜像所依据的容器的id。
    4. sang/nginx则表示仓库名，sang是名称空间，nginx是镜像名。
    5. v1表示仓库的tag。
    6. 创建完成后，通过docker images命令就可以查看到刚刚创建镜像
    例子: 
    docker commit -m "test" --author="lizengcai" 47cb467b3b2f lizengcai/nginx:v1.0
    docker run -itd -p 80:80 --name nginx lizengcai/nginx:v1.0 [/bin/bash]
    
docker build: 从镜像中构建新镜像
docker push username/images[tag]: 推送镜像到docker hub
docker tag source[tag] target[tag]: 修改标签名(由source-->target)
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
RESPOSITORY: 仓库名称,存放同一类型的镜像,有一下几种命名方式
    [namespace\ubuntu]:这种仓库名称由命名空间和实际的仓库名组成，中间通过\隔开。
        当开发者在Docker Hub上创建一个用户时，用户名就是默认的命名空间，这个命令
        空间是用来区分Docker Hub上注册的不同用户或者组织（类似于GitHub上用户名的作用），
        如果读者想将自己的镜像上传到DockerHub上供别人使用，则必须指定命名空间。
        
    [ubuntu]:这种只有仓库名，对于这种没有命名空间的仓库名，可以认为其属于顶级命名空间，
        该空间的仓库只用于官方的镜像，由Docker官方进行管理，但一般会授权给第三方进行开
        发维护。当然用户自己创建的镜像也可以使用这种命名方式，但是将无法上传到Docker Hub上共享
        
    [hub.c.163.com/library/nginx]:这种指定url路径的方式，一般用于非Docker Hub上的镜像命名，
        例如一个第三方服务商提供的镜像或者开发者自己搭建的镜像中心，都可以使用这种命名方式命名。

TAG: TAG用于区分同一仓库中的不同镜像，默认为latest

IMAGE ID: 镜像的唯一标识

CREATED/SIZE:创建时间/大小
```

#### 2.4 docker build总结(从当前镜像中构建新镜像)
- 前提是构建Dockerfile文件
> nginx镜像的构建文件:
> FROM nginx
> MAINTAINER anker "lizengcai_vip@sina.com"
> RUN echo 'hello docker' > /usr/share/nginx/html/index.html
> COPY ./hello.html /usr/share/nginx/html

- docker build命令
> docker build -t anker/nginx:v1 .
> docker run -itd --name nginx -p 80:80 anker/nginx:v1

```php
参数解释:
-t: tag list.格式为name:tag
.: 指的是dockerfile的路径(上下文)

FROM nginx表示该镜像的构建，以已有的nginx镜像为基础，在该镜像的基础上构建。

MAINTAINER指令用来声明创建镜像的作者信息以及邮箱信息，这个命令不是必须的。

RUN指令用来修改镜像，算是使用比较频繁的一个指令了，该指令可以用来安装程序、
    安装库以及配置应用程序等，一个RUN指令执行会在当前镜像的基础上创建一个新的镜像
    层，接下来的指令将在这个新的镜像层上执行，RUN语句有两种不同的形式：shell格式
    和exec格式。本案例采用的shell格式，shell格式就像linux命令一样，exec格式则
    是一个JSON数组，将命令放到数组中即可。在使用RUN命令时，适当的时候可以将多个
    RUN命令合并成一个，这样可以避免在创建镜像时创建过多的层。

COPY语句则是将镜像上下文中的hello.html文件拷贝到镜像中。
```