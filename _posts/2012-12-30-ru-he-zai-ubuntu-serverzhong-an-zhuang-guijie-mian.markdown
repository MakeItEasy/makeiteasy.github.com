---
layout: post
title: "如何在ubuntu server中安装GUI界面"
date: 2012-12-30 13:41
comments: true
tags: [linux, ubuntu]
---

#### 关于ubuntu server

其实个人认为ubuntu server和ubuntu desktop版本没有什么本质的区别，只是默认安装的软件/组件不同而已。
比如ubuntu server默认会安装很多服务，比如mysql，apache等等，而desktop版本会默认安装一些常用的办公软件等。
ubuntu server默认不安装桌面环境，如果需要的话可以按照以下步骤进行安装。

#### 安装GUI步骤  

```
apt-get install xinit
apt-get install gdm
apt-get install ubuntu-desktop
```
