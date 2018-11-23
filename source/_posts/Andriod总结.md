---
title: Andriod总结
date: 2018-11-20 11:08:09
tags:
categories: andriod
---

### 1. andriod进程保活手段
```php
1:利用不同的app进程使用广播来进行相互唤醒

2:它是利用系统的漏洞来启动一个前台的Service进程，与普通的启动方式区别在于，它不会在系统通知栏处出现一个Notification，看起来就如同运行着一个后台Service进程一样。

3:调用系统api启动一个前台的Service进程，这样会在系统的通知栏生成一个Notification，用来让用户知道有这样一个app在运行着，哪怕当前的app退到了后台.比如网易云音乐.
```