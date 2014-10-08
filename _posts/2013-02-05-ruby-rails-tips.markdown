---
layout: post
title: "Ruby,Rails的一些概念和注意点"
date: 2013-02-05 14:39
comments: true
tags: [ruby, rails]
---

#### Ruby,Rails的一些概念

* rvm: ruby version manager, Ruby版本管理工具
* gem(rubygem): 一个ruby程序，用来管理gem包的安装等，类似linux下的apt-get  
	ruby1.9.2以前版本需要 `require 'rubygems'` ，ruby1.9.2开始已经自动包含gem。
* bundle: 用来管理一个rails web工程的所有gem包的依赖，版本等。
* rake: 一个gem包，也就是一个ruby程序，作用是用来执行其他用ruby开发的task的程序。
* rack: A Ruby Webserver interface.是一个提供了一个ruby web服务器和ruby web框架之间的最小接口的gem程序。主要是web框架开发者用的。

#### Ruby,Rails的一些注意点

* .ru后缀的文件就是rackup文件。比如rails应用中的config.ru,可以通过 `rackup config.ru` 来执行rackup文件。
