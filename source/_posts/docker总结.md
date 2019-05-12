---
title: docker总结
date: 2018-12-10 23:00:54
tags:
categories:
- 容器
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

### 3. docker技术入门与实践
#### 3.11 获取镜像
```php
docker [image] pull NAME[:TAG]
    NAME是镜像仓库名称（用来区分镜像），TAG是镜像的标签（往往用来表示版本信息）。 通常情况下，描述一个镜像需要包括“名称＋标签“信息
        例如， 获取一个Ubuntu 18.04 系统的基础镜像可以使用如下的命令：docker pull ubuntu: 18.04
    如果不显式指定TAG,则默认会选择la迳釭标签，这会下载仓库中最新版本的镜像。
        docker pull ubuntu,该命令实际上下载的就是ubuntu:latest镜像。
    image如果为非官方,要加registry(注册地),即registry/image
    使用镜像,在其中运行bash: docker run -it ubuntu:18.04 bash
        echo "Hello World"
        exit
```
> 注意: 一般来说，镜像的latest标签意味着该镜像的内容会跟踪最新版本的变更而变化，内容是不稳定的。因此，从稳定性上考虑，不要在生产环境中忽略镜像的标签信息或使用默认的latest标记的镜像。

#### 3.12 查看镜像信息
```php
docker images/docker image ls:列出本机已有镜像的基本信息
    REPOSITORY: 来自于哪个仓库
    TAG: 标签ID,标签只是标记,并不能标识镜像内容
    IMAGE ID: 镜像ID(唯一标识镜像),两个相同的id说明他们指向同一个镜像,只是具有不同的标签名称而已
    CREATED: 创建时间，说明镜像最后的更新时间
    SIZE: 镜像大小，优秀的镜像往往体积都较小
```

> 注意:
> 其中镜像的ID信息十分重要，它唯一标识了镜像。在使用镜像ID的时候， 一般可以使用该ID的前若干个字符组成的可区分串来替代完整的ID。
> TAG信息用于标记来自同一个仓库的不同镜像。 例如ubuntu仓库中有多个镜像，通过TAG信息来区分发行版本，如18.04、18.10等。
> 镜像大小信息只是表示了该镜像的逻辑体积大小，实际上由于相同的镜像层本地只会存储一份，物理上占用的存储空间会小于各镜像逻辑体积之和。

```php
docker tag image:tag newimage:tag
    添加镜像标签
    例: docker tag ubuntu:18.04 myubuntu:18.04,之后直接使用新的镜像

docker [image] inspect ubuntu:18.04
    获取镜像的详细信息
    获取某一项: docker [image] inspect -f {{.Architecture}} ubuntu:18.04
docker history myubuntu:18.04 [--no-trunc=true]
    查看镜像历史[不截取]
```

#### 3.13 搜寻镜像
```php
docker search --filter=is-official=true nginx
    搜寻官方nginx镜像
docker search --filter=stars=4 tensorflow
    搜寻收藏数超过4的tensorflow关键词
```

> 注(参数):
> -f, --filter filter: 过滤输出内容
> --format string:格式化输出内容；
> --limit int：限制输出结果个数，默认为25个；
> --no-trunc: 不截断输出结果。

#### 3.14 删除镜像
```php
docker rmi/docker image rm IMAGE [IMAGE]
    IMAGE可以为标签或ID
    docker rmi  myubuntu:18.04(通过标签删除镜像)
        本地的ubuntu:18.04没有受到影响,说明只是删除了相同镜像的一个标签副本而已,通过镜像id删除会删除整个镜像文件
        有当前镜像所创建的容器存在时不允许删除镜像,需删除容器,再删除镜像,不推荐使用-f强制删除
    docker image prune -f
        自动清理临时的遗留镜像文件层，最后会提示释放的存储空间
```

#### 3.15 创建镜像
```php
### 方式一:基于已有容器创建
docker [container] commit [OPTIONS] CONTAINER [RESPOSITORY[:TAG]]
    OPTIONS包括:
        -a,--author="":作者信息
        -c,--change=[] : 提交的时候执行Dockerfile指令， 包括CMDIENTRYPOINT但NVIEXPOSEILABELIONBUILDIUSERIVOLUMEIWORKDIR等；
        -m,--message="":提交信息
        -p,--pause=true:提交时暂停容器运行
    例子:
        docker run -it ubuntu:18.04 /bin/bash
            touch test
            exit
        记住此容器id:464d9fbe82bd,此容器与原镜像相比,已经发生了变化,用docker [container] commit命令提交为一个新的镜像
        docker [container] commit -m "Added a new file" -a "Anker Li" 464d9fbe82bd test:0.1
        成功则返回镜像id:f58fec681544353dd202da99ece2ad95921f18adbc2695c86204b7f2a299a134
        docker images查看已经存在test:0.1镜像
### 方式二:基于本地模板导入
docker import [OPTIONS] file|URL|- [RESPOSITORY[:TAG]]
    模板下载地址: http://openvz.org/Download/templates/precreated
    cat ubuntu-18.04-x86_64-minimal.tar.gz | docker import -ubuntu:18.04
### 方式三:基于Dockerfile创建
    最常见的创建镜像方式,看后续介绍
```

#### 3.16存出与载入镜像
```php
docker save -o string [RESPOSITORY[:TAG]]
    导出为本地文件(包括元数据信息,如标签等)
    docker save -o ubuntu_18.04.tar ubuntu:18.04
docker load -i ubuntu_18.04.tar或者docker load < ubuntu_18.04.tar
    导入到镜像
```

#### ##########
#### 3.21 创建容器
```php
docker [container] create [OPTIONS]
    创建容器
    docker create -it ubuntu:latest
    docker ps -a
    注:docker create命令创建的容器处于停止状态,用docker start命令来启动它
    OPTIONS包括(详细请看资料书):
        -d,--detach=true|false:是否在后台运行,默认为否
        -i,--interactive=true|false:保持标准输入打开,默认为否
        -t,--tty=true|false:是否分配一个伪终端,默认为否
        -v,--volume[=[[HOST-DIR:]CONTAINER-DIR[:OPTIONS]]]:挂载临时文件卷到容器内
        -p,--publish=[]:指定如何映射到本地主机端口,例如-p 11234-12234:1234-2234
        --rm=true|false:容器退出后是否自动删除,不能跟-d同时使用
docker start CONTAINER_ID:
    启动容器
docker run CONTAINER_ID:
    创建并启动容器,等价于docker create+start
    运行命令后的后台标准操作(docker run ubuntu:18.04 /bin/echo 'Hello world'):
        检查本地是否存在指定的镜像，不存在就从公有仓库下载；
        利用镜像创建一个容器，并启动该容器；
        分配一个文件系统给容器，并在只读的镜像层外面挂载一层可读写层；
        从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去；
        从网桥的地址池配置一个IP地址给容器；
        执行用户指定的应用程序；
        执行完毕后容器被自动终止。
    启动一个bash终端:
        docker run -it ubuntu:18.04 /bin/bash
        其中，－t选项让Docker分配一个伪终端（pseudo－tty）并绑定到容器的标准输入上，-i则让容器的标准输入保持打开
    启动守护进程运行:
        docker run -d ubuntu /bin/sh -c "while true;do echo hello world; sleep 1;done"
    查看容器的输出:
        docker [container] logs
            OPTIONS包括:
                -details:打印详细信息
                -f,-follow:持续保持输出
                -since string:输出最近的若干日志
                -t,-timestamps:显示时间戳
                -until string:输出某个时间之前的日志
            例如:docker logs c23390f0b2a2
```

#### 3.22 停止容器
```php
### 1.暂停容器
docker [container] pause CONTAINER [CONTAINER...]
    docker run --name test --rm -it ubuntu bash
    docker pause test 
    docker ps
docker [container] unpause CONTAINER [CONTAINER...] 恢复运行状态

### 2.终止容器
docker [container] stop [-t|--time[=10]] [CONTAINER...]:先发送SIGTERM信号,等待一段时间后(默认10s),再发送SIGKILL信号
docker [container] kill [CONTAINER...]:发送SIGKILL信号强行终止容器
docker start [CONTAINER...]:开启容器
docker restart [CONTAINER...]:重启容器
```

#### 3.23 进入容器
```php
### 1.attach命令
docker [container] attach [--detach-keys[=[]]] [--no-stdin] [--sig-proxy[=true]] CONTAINER
    OPTIONS包括:
        --detach-keys[=[]]:指定退出attach模式的快捷键序列,默认是ctrl-p,ctrl-q
        --no-stdin=true|false:是否关闭标准输入,默认是打开
        --sig-proxy=true|false:是否代理收到的系统信号给应用进程，默认为true。
    例子:
    docker run -itd --name test ubuntu
        -i:给ubuntu镜像分配标准能力
        -t:给ubuntu镜像分配伪终端
        -d:后台运行容器，并返回容器ID；
    docker attach test进入终端,ctrl-p,ctrl-q退出终端

    弊端:
    <1> 当多个窗口同时attach到同一个容器的时候，所有窗口都会同步显示
    <2> 当某个窗口因命令阻塞时，其他窗口也无法执行操作了。
### 2.exec命令
docker [container] exec [-d|--detach] [--detach-keys[=[]]] [-i|-interactive] [--privileged] [-t|--tty] [-u|--user[=USER]] CONTAINER COMMAND [ARG...]
    OPTIONS包括:
        -d,--detach:在容器中后台执行命令
        --detach-keys="":指定容器切回后台的按键
        -e,--env=[]:指定环境变量列表
        -i,--interactive=true|false:打开标准输入接受用户输入命令，默认值为false
        --privileged=true|false:是否给执行命令以最高权限,默认为false
        -t,--tty=true|false:分配伪终端,默认值为false
        -u,--user="":执行命令的用户或ID
    例子:
    docker exec -it test /bin/bash
    ctrl-p,ctrl-q退出终端
```

#### 3.24 删除容器
```php
docker [container] rm [-f|--force] [-l|--link] [-v|--volumes] CONTAINER [CONTAINER...]
    OPTIONS包括:
        -f,--force=false:是否强行终止并删除一个运行中的容器
        -l,--link=false:删除容器的连接,但保留容器
        -v,--volumes=false:删除容器挂载的数据卷
    注:
    如果要直接删除一个运行中的容器，可以添加－f参数。Docker会先发送SIGKILL信号给容器，终止其中的应用，之后强行删除
    docker run -d --name test1 ubuntu:18.04 /bin/sh -c "while true;do echo hello world;sleep 1;done"
    docker rm test1 #报错
    docker rm -f test1
```

#### 3.25 导入和导出容器
```php
### 导出容器:
docker [container] export [-o|--output[=""]] CONTAINER
    例子(两种方式):
    docker export -o test.tar ce5
    docker export ce5 > test.tar
### 导入容器:
docker import [-c|--change[=[]] [-m|--message[=MESSAGE]] file|URL|- [REPOSITORY[:TAG]]
    用户可以通过-c,-change=[]选项在导人的同时执行对容器进行修改的Dockerfile指令
    例子:
        docker import test.tar - test/ubuntu:v1.0
        docker images #查看
    docker load(镜像导入)与docker export(容器导入)的区别:
        实际上，既可以使用dockerload命令来导入镜像存储文件到本地镜像库，也可以使用docker[container] import命令来导入一个容器快照到本地镜像库。
        这两者的区别在于：容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积更大。
        此外，从容器快照文件导人时可以重新指定标签等元数据信息
```

#### 3.26 查看容器
```php
### 1.查看容器详情
docker [container] inspect [OPTIONS] CONTAINER [CONTAINER...]

### 2.查看容器内进程信息
docker [container] top [OPTIONS] CONTAINER [CONTAINER...]

### 3. 查看统计信息
docker [container] stats [OPTIONS] [CONTAINER...]
    OPTIONS包括:
        -a,--all:输出所有容器统计信息,默认仅在运行中
        --format string:格式化输出信息
        --no-stream:不持续输出,默认会自动更新实时结果
        --no-trunc:不截断输出结果。
```

#### 3.27 其他容器命令
```php
### 1. 容器与主机间复制文件
docker [container] cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
    OPTIONS包括:
        -a,--archive:打包模式,复制文件会带有原始的uid/gid信息
        -L,--follow-link:跟随软连接,当原路径为软连接时,默认只复制链接信息，使用该选项会复制链接的目标内容。
    例子(将本地路径的data复制到test容器的/tmp路径下):
        docker [container] cp data test:/tmp/

### 2. 查看容器内文件变更
docker [container] diff CONTAINER
    例子:
        docker container diff test

### 3. 查看容器端口映射情况
docker [container] port CONTAINER [PRIVATE_PORT[/PROTO]]
    例子:
        docker container port test

#### 4. 更新配置(后续更新)
```

#### ##########
#### 3.31 数据卷
```php
定义: 
    数据卷（DataVolumes）是一个可供容器使用的特殊目录，它将主机操作系统目录直接映射进容器，类似于Linux中的mount行为。
特性:
    数据卷可以在容器之间共事和重用，容器间传递数据将变得高效与方便；
    对数据卷内数据的修改会立马生效，无论是容器内操作还是本地操作；
    对数据卷的更新不会影响镜像，解耦开应用和数据；
    卷会一直存在，直到没有容器使用，可以安全地卸载它
### 1.创建数据卷
docker volume create -d local test
docker inspect/ls/prunu/rm:查看,列出,清除无用,删除数据卷

### 2.绑定数据卷
在运行docker run命令时,可以使用--mount选项来使用数据卷
--mount支持三种数据卷:
    volume: 普通数据卷,映射到主机/var/lib/docker/volume路径下
    bind: 绑定数据卷,映射到主机指定目录下
    tmpfs:临时数据卷,只存在于内存中
例子:
    docker run -d -P --name web mount type=bind,source=/webapp,destination=/opt/webapp training/webapp python app.py
类似: 
    docker run -d -P --name web -v /webapp:/opt/webapp training/webapp python app.py
    Docker挂载数据卷的默认权限是读写（rw），用户也可以通过ro指定为只读：
    docker run -d -P --name web -v /webapp:/opt/webapp:ro training/webapp python app.py
```

#### 3.32 端口映射与容器互联
```php
### 1. 端口映射
当使用-P（大写的）标记时，Docker会随机映射一个49000-49900的端口到内部容器开放的网络端口
    docker run -d  -P  training/webapp python app.py
    使用 docker ps  -l 命令找到容器名称为nostalgic_morse
    docker logs  -f nostalgic_morse
-p (小写的）则可以指定要映射的端口，并且，在一个指定端口上只可以绑定一个容器。支持的格式有 IP:HostPort:ContainerPort|IP:: ContainerPort|HostPort:ContainerPort
    解释:
    HostPort:ContainerPort(本地5000到容器5000,3000到80)
        docker run -d -p 5000:5000 -p 3000:80 training/webapp python app.py
    IP:HostPort:ContainerPort(本地5000到容器5000)
        docker run -d -p 127.0.0.1:5000:5000 training/webapppython app.py
    IP:: ContainerPort(本地随机分配端口到容器5000)
        docker run -d -p 127.0.0.1::5000 training/webapp python app.py
查看容器端口配置:
    docker port m_nginx
查看容器特定端口配置:
    docker port m_nginx 8080
### 2. 容器互联
例子:
    docker run \
      -d \
      -p 80:80 \
      -p 8080:8080 \
      -p 8081:8081 \
      -v /data/wwwroot:/usr/share/nginx/html \
      -v /data/nginx/nginx.conf:/etc/nginx/nginx.conf:ro \
      -v /data/nginx/conf:/etc/nginx/conf.d \
      -v /data/wwwlogs:/var/log/nginx \
      --link m_phpfpm:phpfpm \
      --name m_nginx nginx:latest
解释:
    --link的参数格式为:--link name: alias,其中name是要链接的容器的名称，alias是别名。在m_nginx容器内可以看到对m_phpfpm的使用:
        fastcgi_pass   phpfpm:9000;
Docker通过两种方式为容器公开连接信息：
    1. 更新环境变量,在m_nginx容器内通过env命令可以看到添加了m_phpfpm的变量信息,如: PHPFPM_NAME,PHPFPM_PORT_9000_TCP_PROTO,PHPFPM_PORT等
    2. 更新/etc/hosts文件,m_nginx容器内的hosts: 
        172.17.0.3  phpfpm 24bbbd9c5ab7 m_phpfpm
        172.17.0.4  fa6bc439949f
```
#### ##########
#### 3.41 使用Dockerfile创建镜像


### 5. 实例(PHP+Nginx+Mysql环境)
#### 5.11 创建资源文件夹
```php
/data
    /mysql  存放数据库备份
    /nginx  nginx配置文件
        /conf
        /rewrite
    /wwwroot  网站文件
    /wwwlogs  日志
    /source  程序源代码
    /download 下载目录
```

#### 5.12 部署mysql
1. 拉取镜像
```php
docker pull mysql:5.7
```
2. 运行容器
```php
docker run \
-d \
-p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=12345678910 \
--name m_mysql mysql:5.7
```

> 参数说明
> -d 让容器在后台运行
> -p 添加主机到容器的端口映射，前面是映射到本地的端口，后面是需要映射的端口
> -e 设置环境变量，MYSQL_ROOT_PASSWORD这里是设置mysql的root用户的初始密码
> --name 容器的名字，随便取，但是必须唯一

3. 进入容器
```php
docker exec -it m_mysql /bin/bash
### 安装vim
apt-get update
apt-get install vim
```
> 参数说明
> -t 在容器里生产一个伪终端
> -i 对容器内的标准输入 (STDIN) 进行交互

4. 开启Mysql远程连接
```php
### 1. 进入mysql
mysql -uroot -p12345678910 
### 2. 选择数据库
use mysql
### 3.开启远程连接
> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;
> FLUSH PRIVILEGES;
### 4. 重启
service mysqld restart
```

#### 5.13 部署php
1. 拉取镜像
```php
docker pull bitnami/php-fpm:7.0
```
2. 运行容器
```php
docker run \
  -d \
  -p 9000:9000 \
  -v /data/wwwroot:/usr/share/nginx/html \
  --link m_mysql:mysql \
  --name m_phpfpm bitnami/php-fpm:7.0
```
> 参数说明
> -d 让容器在后台运行
> -p 添加主机到容器的端口映射
> -v 添加目录映射
> –-name 容器的名字，随便取，但是必须唯一
> --link link 是在两个contain之间建立一种父子关系，父container中的web，可以得到子container db上的信息。
> 通过link的方式创建容器，我们可以使用被Link容器的别名进行访问，而不是通过IP，解除了对IP的依赖。

3. 创建php文件
```php
### 在/data/wwwroot文件夹中创建一个文件夹default，代表一个默认的工作目录，在default中创建一个文件
touch index.php
### 编辑php文件
<?php
  echo "hello!";
?>
### 这时候你进php容器，在/usr/share/nginx/html文件夹中也能看到我们创建的index.php文件，因为我们设置了目录映射。这样我们就可以直接在宿主机直接更改，而不用进容器修改
```

#### 5.14 配置nginx
1. 拉取镜像
```php
docker pull nginx
```
2. 运行容器
```php
### 1. 我们先运行无挂载的容器，拿到nginx的一些配置文件
docker run \
  -d \
  -p 80:80 \
  --name m_nginx nginx:latest

### 2. 然后使用docker cp操作复制出文件
docker cp m_nginx:/etc/nginx/nginx.conf /data/nginx
docker cp m_nginx:/etc/nginx/conf.d/default.conf /data/nginx/conf

### 3. 删除之前的容器
docker stop m_nginx
docker rm m_nginx

### 4. 重新运行容器
docker run \
  -d \
  -p 80:80 \
  -p 8080:8080 \
  -p 8081:8081 \
  -v /data/wwwroot:/usr/share/nginx/html \
  -v /data/nginx/nginx.conf:/etc/nginx/nginx.conf:ro \
  -v /data/nginx/conf:/etc/nginx/conf.d \
  -v /data/wwwlogs:/var/log/nginx \
  --link m_phpfpm:phpfpm \
  --name m_nginx nginx:latest
```

> 参数说明：
> -d 让容器在后台运行
> -p 添加主机到容器的端口映射
> -v 添加目录映射,这里最好nginx容器的根目录最好写成和php容器中根目录一样。但是不一点非要一模一样,如果不一样在配置nginx的时候需要注意
> -–name 容器的名字
> –-link 与另外一个容器建立起联系

3. 编辑配置文件
```php
### 修改/data/nginx/conf/default.conf，我的配置如下：

server {
    listen       80;
    server_name  _;
    #charset koi8-r;
    access_log  /var/log/nginx/default_nginx.log  main;
    location / {
        root   /usr/share/nginx/html/default;
        index  index.html index.htm;
    }
    #error_page  404              /404.html;
    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}
    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    location ~ \.php$ {
        root           /usr/share/nginx/html/default;
        fastcgi_pass   phpfpm:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

#### 5.15 安装phpMyAdmin
1. 下载解压
下载地址: https://www.phpmyadmin.net/
解压到/data/wwwroot/default，并将文件夹重命名为phpMyAdmin


2. 配置phpMyAdmin
打开 libraries 目录下，找到config.default.php,用文本编辑器打开文件。
在blowfish_secret中填入值，如：
```php
$cfg['blowfish_secret'] = 'sadsadsadasffdsadfsa'; /* YOU MUST FILL IN THIS FOR COOKIE AUTH! */
```
在host中填入数据库的访问IP，这里我填的Mysql容器的内网IP，你也可以填外网IP试试。
```php
$cfg['Servers'][$i]['host'] = '172.17.0.2';
```

3. 访问http://外网IP/phpMyAdmin/index.php












































