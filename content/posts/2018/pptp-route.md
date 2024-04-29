---
title: 如何让pptp连接/断开后自动添加路由

date: 2018-10-19

categories: 技术分享

---


测试平台：Openwrt


在```/etc/hotplug.d/iface/```下添加```30-routes```文件,系统将在接口状态发生生变化时执行该脚本。

<!-- more -->

然后在```30-routes```中写入


```
#!/bin/sh

if [ "$ACTION" = "ifdown" -a "$INTERFACE" = "pptp-vpn0" ]
then

#在此添加断开后的代码

fi

if [ "$ACTION" = "ifup" -a "$INTERFACE" = "pptp-vpn0" ]
then

#在此添加连接后需要执行的代码

fi
```


把```pptp-vpn0```修改为自己想要的接口名。


比如我现在需要在```pptp-vpn0```断开后自动添加192.168.0.1为默认路由，则设置如下


```
if [ "$ACTION" = "ifdown" -a "$INTERFACE" = "virtual**" ]
then

route add default gw 192.168.0.1

fi
```


加入了一行```route add default gw 192.168.0.1```
