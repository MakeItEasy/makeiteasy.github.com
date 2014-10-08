---
layout: post
title: "我在调查jpmobile过程中的总结"
date: 2013-11-12 13:27
comments: false
tags: [rails]
---

#### 概述

JPMobile([Github](https://github.com/jpmobile/jpmobile))，是一个rails的gem包，主要是用来简化针对日本不同的手机客户端view层的设计。  
Ripple是一个chrome的plugin，可以模拟很多手机的浏览器。

#### 关于ruby的一些小知识

* Module.autoload is a instance_method  
* Kernel.autoload is not a instance_method  
* module\_function : 定义模块方法  
* :: 引用顶级命名空间，例子 ::File  
* `/^(?:abcde)/` 其中`?:`的意思是:只匹配exp,不捕获匹配的文本，也不给此分组分配组号。

#### 有用的链接
* [userAgent（ユーザーエージェント一覧）](http://www.openspc2.org/userAgent/)  
* [关于ripple不会发送mobile本身的user agent的过渡方案](https://github.com/blackberry/Ripple-UI/issues/614)  
上文中提到的可能要用到的路径：`C:\Users\dairg\AppData\Local\Google\Chrome\User Data\Default\Extensions\cnijnnaimeaacneklcndcafbnkeicckh\0.9.13_0\controllers`  

