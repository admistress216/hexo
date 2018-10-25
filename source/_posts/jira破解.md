---
title: jira破解
date: 2018-07-05 11:36:38
tags:
---

- <i>注:软件包在微云</i>
<br/>

### 1. 安装java环境

```php
# 安装jdk
rpm -ivh jdk-8u131-linux-x64.rpm

# 检测是否安装成功
java -version

# 出现以下信息代表安装成功
# java version "1.8.0_131"
# Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
# Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode)
```

### 2. 安装mysql5.6
<br/>
- 安装教程见[github](https://github.com/anker1992/www/blob/master/mysql/mysql.php)
```php
# 注意: 安装完成后创建jira数据库
create database jira default character set 'utf8' collate 'utf8_bin';
```

### 3. 下载jira

```php
# 解压
tar zxvf atlassian-jira-software-7.3.6.tar.gz
# 移动
mv atlassian-jira-software-7.3.6-standalone /usr/local/
```

### 4. 配置jira_home

```php
mkdir /var/jira 

vi ~/.bash_profile
文件最后一行增加 export JIRA_HOME=/var/jira

同时在终端也输入以下命令
export JIRA_HOME=/var/jira
```

### 5. 给jira安装mysql驱动com.mysql.jdbc.Driver

```php
tar zxvf mysql-connector-java-5.1.42.tar.gz
将mysql-connector-java-5.1.42-bin.jar复制到jira的安装目录下的confluence/WEB-INF/lib/目录
cp mysql-connector-java-5.1.42/mysql-connector-java-5.1.42-bin.jar /usr/local/atlassian-jira-software-7.3.6-standalone/atlassian-jira/WEB-INF/lib/
```

### 6. 启动jira

```php
cd /usr/local/atlassian-jira-software-7.3.6-standalone/bin
./start-jira.sh
启动jira,一般需要2分钟才能启动。
```

### 7. jira配置license
- 通过网址访问jira,自定义配置.

### 8. 破解jira

```php
unzip jira7.3.zip
# 将包中的atlassian-extras-3.2.jar 复制到 usr/local/atlassian-jira-software-7.3.6-standalone/atlassian-jira/WEB-INF/lib/ 替换原有的包
# 重启jira
/usr/local/atlassian-jira-software-7.3.6-standalone/bin/stop-jira.sh
/usr/local/atlassian-jira-software-7.3.6-standalone/bin/start-jira.sh
```

### 9. nginx反向代理(记得加gzip)

```php

server {
        listen 80;
        server_name jira.newscctv.net;

        location /{
                proxy_set_header X-Forwarded-Host $host;
                proxy_set_header X-Forwarded-Server $host;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_pass http://192.168.201.35:8080;  # 修改成jira服务器的IP
                client_max_body_size 10M;
        }

}

```

### 10. tomcat配置修改(保证域名一致性,而不是有些地方是ip)
```php
<Service name="Catalina">

        <Connector port="8080"

                   maxThreads="150"
                   minSpareThreads="25"
                   connectionTimeout="20000"

                   enableLookups="false"
                   maxHttpHeaderSize="8192"
                   protocol="HTTP/1.1"
                   useBodyEncodingForURI="true"
                   redirectPort="8443"
                   acceptCount="100"
                   disableUploadTimeout="true"
                   proxyName="jira.newscctv.net"  
                   proxyPort="80"
                   bindOnInit="false"/>
```

> 位置: /usr/local/atlassian-jira-software-7.3.6-standalone/conf/server.xml
> 添加proxyName="jira.newscctv.net"  proxyPort="80"