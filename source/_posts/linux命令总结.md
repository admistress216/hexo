---
title: linux命令总结
date: 2018-07-27 15:06:34
tags:
---

```php
//1. 重启网络
/etc/rc.d/init.d/network restart
//2. 网关地址查询
netstat -rn //r:display route table(显示路由表)
ip route show
//3. 添加网关
route add default gw 10.211.55.254
//4. 设置ip和子网掩码
ifconfig eth0 10.211.55.6 netmask 255.255.255.0
//5. 修改dns
echo "nameserver 202.202.202.20" >> /etc/resolv.conf
```