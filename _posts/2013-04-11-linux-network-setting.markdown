---
layout: post
title: "linux下网络设置"
date: 2013-04-11 17:26
comments: true
categories: [linux]
---

#### Ubuntu系统

* 网络配置文件路径: `/etc/network/interfaces`  
* 路由设置配置文件路径: `/etc/resolv.conf`  
    注意有时候经常设置固定ip后上不了网，这个时候可能是因为dns配置没有修改过来。  

#### CentOS系统

* 网络配置文件路径:  `/etc/sysconfig/network-scripts/ifcfg-eth0`  
* 路由设置配置文件路径: `/etc/resolv.conf` , 这个文件手动改不管用，会自动改回去，可以在上面的配置中加入DNS的配置，重启网络就可以自动重新配置这个文件。


#### 特别注意

如果要在windows端手动的修改vmnet8的ip地址等，那么修改之后如果还需要让虚拟机访问外网，那么需要针对能够上外网的网卡进行共享配置。
比如现在是local network有限上网，那么就需要在本地网络的属性中有[共有]那个tab里面要把本地网络共享给vmnet8，这样的话，才可以在
虚拟机里面上外网。
