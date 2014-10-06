---
layout: post
title: "linux常用命令"
date: 2012-12-31 23:47
comments: true
categories: [linux, moyan manual]
---

#### 桌面命令类

* 切换到不同tty的命令 `chvt 1-7`

#### 实用命令类

* 查看当前系统发行版的信息 `lsb_release`
* 查看系统硬件信息目录 `/proc`
* 文本处理三剑客 `grep` `sed` `awk`
	1. grep:主要是查找字符串
	2. sed:主要是以行为单位处理字符串
	3. awk:主要是针对一行字符串，利用间隔符，对每一段进行处理
* 列出所有打开的文件，可以进行各种网络调试等。 `lsof`
* 强制kill: `kill -9 1234`

#### 设置类

* 查看可用分辨率 `xrandr`
* 设置分辨率 `xrandr -s 可用分辨率的具体数字`
* 输出要添加到xorg文件中section minitor的内容 `gtf 1024 768 70`, 参数是分辨率和刷新频率
* 生成xorg文件 `xorg -configure`, 拷贝到 `/etc/X11/xorg.conf`


#### 好的参考资源

* [一篇很好的讲到linux软连接和硬链接的文章](http://www.ibm.com/developerworks/cn/linux/l-cn-hardandsymb-links/)
