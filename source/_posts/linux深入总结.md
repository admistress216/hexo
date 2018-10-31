---
title: linux深入总结
date: 2018-10-03 03:44:50
tags:
---

### 1. 编程层次
硬件规格(Hardware Specification)
系统调用(System Call)
库调用(Library Call)

### 2. user interface(UI):用户接口
GUI:Graphic User Interface(图像用户接口)
CLI:Command Line Interface(命令行接口)
ABI:Application Binary Interface(运行接口):程序编译成二进制可执行文件所使用的接口
API:Application Programing Interface(编程应用接口):比如程序未编译前所使用的接口

### 3.书籍
<<奇点临近>>

### 4.运行程序格式:
	Windows:EXE,dll(dynamic link library)
	Linux:ELF,so(shared object)

### 5.Linux发行版
	slackware
		suse
			opensuse
	debian
		ubantu
			mint
	redhat
		rhel:redhat enterprise linux
		每18个月发行一次新版本
		CentOs:兼容rhel版本
		fedora:测试新软件,每6个月发型一个新版本
	ArchLinux
	Gentoo:更好的发挥硬件性能,比较难
	LFS:Linux From Scratch(linux book):linux怎么制作教程

	Android:kernel+busybox+java虚拟机

### 6.问题
#### 6.1 CentOS和Linux是什么关系?CentOs和RHEL是什么关系?
#### 6.2 各种开源协议的具体细节?
	GPL,LGPL,Apache,BSD

### 7.程序包管理器:
	rpm:
		RHEL,Fedora,S.u.E.E,Centos
	dpt:
		Debian,Ubuntu
		
### 6.概念解释
	GNU:gnu is not unix
	gpl:gnu general public lisence自由软件许可证
	其他协议:apache,bsd,比gpl好

### 7.获取发行版本:
	mirrors.aliyun/souhu/163.com

### 8.打开图形界面
	startx &

### 9.命令
echo $SHELL: 查看当前使用shell
cat /etc/shells: 查看系统支持shell
type command: 区别内部或外部命令

#### 9.1 hash命令
hash: 缓存使用的命令路径于hash表,提高命令的调用速率
hash [-l]: 显示hash表内的内容
hash (-d name)/-r: 清除单个/全部缓存命令
注:
在命令已缓存的情况下,移动命令路径,会出现找不到命令的错误(即使命令在path中),解决方法: 重新登录终端或者删除缓存命令

#### 9.2 帮助命令
```php
# 内部命令
help command

# 外部命令
<1> command --help/-h
<2> 使用手册: man command
<3> 信息页: info command
<4> 程序自身帮助文档(一般在/usr/share/doc/COMMAND-VERSION): README,INSTALL,ChangeLog
<5> 程序官方文档(Documentation)
<6> 发行版的官方文档(http://www.redhat.com/docs)
<7> google.cn
    (openstack filetype:pdf):搜索全部为pdf格式的openstack
    (openstack site:openstack.com):指明站点搜索
<8>(http://www.slideshare.net):pdf/ppt教程
```

#### 9.3 man帮助详解
```php
man COMMAND

// 手册页所在位置
/usr/share/man(大多数手册,具体位置在/etc/man.config中的MANPATH定义)

// man1.....man8介绍
man1: 用户命令
man2: 系统调用
man3: C库调用
man4: 设备文件及特殊文件
man5: 配置文件格式
man6: 游戏
man7: 杂项
man8: 管理类命令

man -M path command   //在指定目录下查找命令,如果目录为空,效果和没有-M参数一样

//man命令段落说明
SYNOPSIS(概要):
    []: 可选内容
    <>:必选内容
    a|b:二选一
    ...:同一内容可多次出现
```
- 注意:有些命令在不知一个章节中存放帮助手册(whereis read),要查看指定章节中的手册: man # command

#### 9.4 history命令
```php
# 命令文件存放于~/.bash_history中,history命令显示的记录(cache中)会在logout时写入~/.bash_history
history -a: 将缓存中的命令追加于文件中
history -d offset: 删除offset处的命令(cache中)
history -c: 删除所有(cache中)
history #: 显示历史中最近#条命令

!#: 调用历史中第#条命令
!command: 调用历史中以command开始的命令
!$/esc, .: 调用上次命令的最后一个参数

控制命令历史的记录方式:
    环境变量: HISTCONTROL
        ignoredups: 忽略重复的命令,连续输入相同命令
        ignorespace: 忽略所有以空白开头的命令
        ignoreboth: ignoredups,ignorespace
    修改环境变量值: export 变量名="值",例子:HISTCONTROL="ignoreboth"
        把赋值符号后面的数据存储于变量名指定的内存空间,所以重新登录会失效
```
#### 9.5 date命令(date --help)
```php
# date [选项]... [+格式]: 显示日期
date +%a/A:显示星期几
date +%D/F: 显示日期
date +%T: 显示时间

# date [-u|--utc|--universal] [MMDDhhmm[[CC]YY][.ss]]: 设定日期
date 102315112018.59

# hwclock - query and set the hardware clock(man hwclock)
-s/-w: 系统时钟与硬件时钟时间互转
```

#### 9.6 相关环境变量
```php
PWD: 保存了当前目录路径(echo $PWD | pwd)
OLDPWD: 上次路径(cd -)
```

#### 9.7 stat + filename: 显示文件元数据信息(size,various time etc.)

#### 9.8 file + filename: 判断文件类型

### 10. 文件系统
#### 10.1 分析

> 系统内核先加载根所在盘,其他盘分别挂载在跟分区的目录上

#### 10.2 各目录的作用
```php
/boot: 引导文件存放目录,内核文件(vmlinuz),引导加载器(bootloader,grub)都存放此目录
/bin: 供所有用户使用的基本命令,不能关联至至独立分区,os启动会用到的程序
/sbin: 管理类的基本命令,不能关联至独立分区,os启动即会用到的程序
/lib: 基本共享库文件,以及内核模块文件(/lib/modules)
/lib64: 专用于x86_64系统上的辅助共享库文件存放位置
/etc: 配置文件目录(纯文本文件)
    /etc/opt: 为/opt提供配置文件
    /etc/x11: 图像系统配置文件
/home: 普通用户目录
/root: 管理员的家目录
/media: 便携式移动设备挂载点
    /media/cdrom: 光盘
    /media/usb: ubs
/mnt: 挂载临时文件系统
/dev: 设备文件及特殊文件存放位置
    b: 块设备(block),随机访问
    c: 字符设备(character),线性访问
/opt: 第三方应用程序的安装位置
/tmp: 临时文件存放位置
/usr: universal shared, read-only data
    bin: 应用程序
    sbin: 应用程序
    lib/lib64:库文件
    include: C程序的头文件
    share: 结构化独立数据
    local: 第三方应用程序的安装目录
        bin,sbin,lib,lib64,etc,share
/var: variable data files
    cache: 应用程序缓存数据目录
    lib: 应用程序状态信息数据
    local: 专用于为/usr/local下的应用程序存储可变数据
    lock: 所文件
    log: 日志目录及文件
    opt: 专用于为/opt下的应用程序存储可变数据
    run: 运行中的进程相关的数据,常用于存储进程的pid文件
    spool: 应用程序数据池
    tmp: 保存系统两次重启之间产生的临时数据
/proc: 用于输出内核与进程信息相关的虚拟文件系统
/sys: 用于输出当前系统上硬件设备相关信息的虚拟文件系统
/selinux: security enhanced Linux,相关安全策略等信息存储位置
```
> 注: 除了/boot和/home,/usr之外几乎不可以独立分区

#### 10.3 Linux上的应用程序的组成部分
```php
二进制程序: /bin, /sbin, /usr/bin, /usr/sbin, /usr/local/bin, /usr/local/sbin
库文件: /lib, lib64, /usr/lib, /usr/lib64, /usr/local/lib, /usr/local/lib64
配置文件: /etc, /etc/Directory, /usr/local/etc
帮助文件: /usr/share/man, /usr/share/doc, /usr/local/share/man, /usr/local/share/doc
```

#### 10.4 关机或重启命令
halt,poweroff,reboot
    -f: 强制,不调用shutdown
    -p: 切断电源
shutdown [OPTION].... TIME [MESSAGE]
    -r: reboot
    -h: halt
    -c: cancel
        TIME: 
            now/+m: +3:3分钟之后
            hh:mm: 指定时间

#### 10.5 登录信息查看
whoami: 显示当前登录有效用户
who: 系统当前所有的登录会话
w: 系统当前所有的登录会话及所做的操作





































































































