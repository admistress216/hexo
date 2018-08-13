---
title: uml教程
date: 2018-07-04 14:17:59
tags:
---

### 1.uml概述
- **UML(Unified Modeling Language)概念:** 统一建模语言,是一种编制软件蓝图的标准化语言;
- **UML为什么很重要:**  写软件就像建筑一样,系统越复杂,完成编写与架构任务的工程师间的交流也就越重要,uml提供了美工,设计师和程序猿之间的通用语言;
<br/>

---------------

### 2.uml模型和特点
#### uml模型主要由三部分构成:
- **事物(Things):** uml构成的基本元素
- **关系(Relationships):** 关系将事物紧密联系在一起
- **图(Diagrams):** 事物与关系的可视化表现

#### uml特点:
- **面向对象**
- **可视化**
- **独立于过程**
- **独立于程序设计**
- **容易掌握**
<br/>

--------------

### 3.PlantUml语法
#### 3.1 [官方文档地址](http://plantuml.com/)
#### 3.2 基础
- @startuml,@enduml
开始结束标记,表示uml解析的部分
- start,end
表示图示的开始和结束
- :Hello world;
活动标签(activity label)以冒号开始，以分号结束。活动默认安装它们定义的顺序就行连接。


---------------

### 4.PlatUml分类及例子
#### 4.1 时序图
``` php
@startuml
Alice -> Bob: Authentication Request
Bob --> Alice: Authentication Response
Alice -> Bob: Another authentication Request
Alice <-- Bob: another authentication Response
@enduml
```

<img src="http://resource.cmdapps.com/2018/07/sequence-diagram-huz1dykz.png" style="" />

> 事物:
> - actor
> - boundary
> - control
> - entity
> - database
> - collections
>
> 说明:
> 第二选项为名称,可以通过as缩写,第三项为排序,第四项为颜色指定
> 使用非字母符号,如:(),\n等,需用""来包含

``` php
@startuml
actor Foo1 #blue
boundary Foo2 order 50
control Foo3 order 60
entity Foo4
database Foo5
collections Foo6
participant Alice
participant "I have a really\nlong name" as L order 80 #99FF99

Foo1 -> Foo2 : To boundary
Foo1 -> Foo3 : To control
Foo1 -> Foo4 : To entity
Foo1 -> Foo5 : To database
Foo1 -> Foo6 : To collections
Foo1 -> Alice : To participant
Foo1 --> L : To abbreviation
L -> L : This is a signal to self.\nIt also demonstrates\nmultiline \ntext

@enduml
```
<img src="http://resource.cmdapps.com/2018/07/sequence-diagram-mejytip4.png" style="width:500px;height:500px" />

> 改变箭头的方式:
> 表示一条丢失的消息：末尾加 x
> 让箭头只有上半部分或者下半部分：将<和>替换成\或者 /
> 细箭头：将箭头标记写两次 (如 >> 或 //)
> 虚线箭头：用 -- 替代 -
> 箭头末尾加圈：->o
> 双向箭头：<->

```php
@startuml
Bob ->x Alice
Bob -> Alice
Bob ->> Alice
Bob -\ Alice
Bob \\- Alice
Bob //-- Alice

Bob ->o Alice
Bob o\\-- Alice

Bob <-> Alice
Bob <->o Alice

@enduml
```
<img src="http://resource.cmdapps.com/2018/07/sequence-diagram-mejytip5.png" style="" />

> 编号:
> autonumber 'start' 'increment'''
> 可以指定编号格式,见官方文档

```php
@startuml
autonumber
Bob -> Alice : Authentication Request
Bob <- Alice : Authentication Response

autonumber 15
Bob -> Alice : Another authentication Request
Bob <- Alice : Another authentication Response

autonumber 40 10
Bob -> Alice : Yet another authentication Request
Bob <- Alice : Yet another authentication Response

@enduml
```
<img src="http://resource.cmdapps.com/2018/07/sequence-diagram-mejytip6.png" style="width:400px;height:400px" />
<center><h1>(.......未完待续......)</h1></center>

#### 4.2 活动图
> 活动标签以冒号(:)开始分号(;)结束
> 可以使用关键字start和stop表示图示的开始和结束

```php
@startuml 
:Hello world;
:This is on defined on
several **lines**;
@enduml
```
<img src="http://resource.cmdapps.com/2018/07/sequence-diagram-mejytip7.png" style="" />

```php
@startuml
start
:Hello world;
:This is on defined on
several **lines**;
stop

start
:Hello world;
:This is on defined on
several **lines**;
end

@enduml
```
<img src="http://resource.cmdapps.com/2018/07/sequence-diagram-mejytip8.png" style="" />

> 在图示中可以使用关键字if，then和else设置分支测试。标注文字则放在括号中
> 也可以使用关键字elseif设置多个分支测试

```php
@startuml
start

if (Graphviz installed?) then (yes)
  :process all\ndiagrams;
else (no)
  :process only
  __sequence__ and __activity__ diagrams;
endif

stop

start
if (condition A) then (yes)
  :Text 1;
elseif (condition B) then (yes)
  :Text 2;
  stop
elseif (condition C) then (yes)
  :Text 3;
elseif (condition D) then (yes)
  :Text 4;
else (nothing)
  :Text else;
endif
stop

@enduml
```
<img src="http://resource.cmdapps.com/2018/07/sequence-diagram-mejytip9.png" style="" />

> 重复循环:
> 使用关键字repeat和repeatwhile进行重复循环

```php
@startuml
start

repeat
  :read data;
  :generate diagrams;
repeat while (more data?) is (not empty)

stop

@enduml
```
<img src="http://resource.cmdapps.com/2018/07/repeat.png" style="" />

> while循环:
> 使用关键字while和endwhile进行while循环
> 可以在关键字endwhile后添加标注，还有一种方式是使用关键字is

```php
@startuml
start

while (data available?)
  :read data;
  :generate diagrams;
endwhile

stop

while (check filesize?) is (not empty)
    :read file;
endwhile (empty)
    :close file;

@enduml
```
<img src="http://resource.cmdapps.com/2018/07/while.png" style="" />

> 并行处理
> 可以使用关键字fork，fork again和end fork表示并行处理

```php
@startuml

start

if (multiprocessor?) then (yes)
  fork
	:Treatment 1;
  fork again
	:Treatment 2;
  end fork
else (monoproc)
  :Treatment 1;
  :Treatment 2;
endif

@enduml
```
<img src="http://resource.cmdapps.com/2018/07/fork.png" style="" />

