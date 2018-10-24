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

!#: 调用历史中第#条命令
!command: 调用历史中以command开始的命令
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













































































































