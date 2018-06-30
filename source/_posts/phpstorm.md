---
title: phpstorm
date: 2018-06-27 10:11:38
categories: php总结 
---

### 1. 目的
记录和练习的目的,旨在提升工作效率,加快开发节奏与流畅度.

### 2. 常用快捷键
  
编辑快捷键
  
> command + / : 添加代码注释//
> command + alt + / : 添加代码注释/* */
> control + alt + i : 快速调整缩进
> command + shift + v : 在剪切板选择粘贴
> command + d : 复制当前行
> command + delete : 删除当前行
> control + shift + j : 把下一行缩上来变成一行
> shift + enter/alt + command + enter : 快速向下/上折行
> command + shift + u : 大小写快速切换
> command + shift + [/] : 选项卡快速切换
> command + +/- : 代码块展开/收缩
> command + shift + f : find in path
> command + f/r : find/replace
> command + delete : delete one line
> alt + / : 代码补全
> command + , : open setting
> command + j : live templates
> control + `(反引号) : switcher1
> control + tab : switch2


导航快捷键:
  
> command + o : 快速查找类
> command + shift + o : 快速查找文件名
> alt + f7 : 找到继承该接口或者父级 的所有子类, 统计所有子类个数(find usage)
> command + l : 定位行,跳转行
> command + e : recent file
> control + j : 获取函数或变量的相关信息
> command + y : 浮窗显示变量或函数声明信息
> f2/shift f2 : 切换到上一个/下一个错误位置
> alt + command + <-/-> : 上部/下部文件操作处
> f3/alt + f3 : 添加标签/带助记的标签
> command + f3 : 打开书签列表


搜索和替换:
  
> command + f : 当前文件搜索
> command + shift + f : 按路径搜索
  
> command + r : 当前文件替换
> command + shift + r : 按路径替换
  
> command + g : 下一个
> command + shift + g : 下一个

全局快捷键:
  
> 双击shift : 全局查找搜索
> command + 1,2,3,4,5 : 打开各种工具窗口

### 3. 常用插件
  
#### <1>live edit : 配合chrome,实现同步编辑(需要chrome安装JetBrains IDE Support插件)
#### <2>ideaVim : 开启vim模式的编辑器(tools->emulator),快捷键:alt+command+v
#### <3>plantuml : 画图(other setting中配置graphviz插件[http://www.graphviz.org/],最好用命令安装brew install graphviz) 
  
  
for example:
```php
@startuml

digraph test {
    label = "测试一下下"
    start [label="开始游戏" shape=circle style=filled]
    ifwifi [label="判断是否是wifi" shape=diamond]
    needupdate [label="是否有资源需要更新" shape=diamond]
    startslientdl[label="静默下载" shape=box]
    enterhall[label="进入游戏大厅" shape=box]


    ' direction
    start -> ifwifi
    ifwifi -> needupdate[label="是"]
    needupdate -> startslientdl[label="是"]
    startslientdl -> enterhall
    needupdate -> enterhall[label="否"]
    ifwifi -> enterhall[label="否"]
}

@enduml

```
<img src="http://resource.cmdapps.com/uml.jpeg" style="width:300px;height:400px" />







































