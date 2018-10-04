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


