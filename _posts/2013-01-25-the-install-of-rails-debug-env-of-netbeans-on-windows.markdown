---
layout: post
title: "WindowsでNetBeansでrailsのDebug環境の構築(失敗した)"
date: 2013-01-25 15:36
comments: true
categories: [rails, env install]
---

#### 前言

这里只是单纯的记录了自己在windows中为NetBeans构筑Debug环境过程中的一些
注意事项，主要是碰到一些错误。并且在windows这次没有成功。  
特别注意: _**这里不是手顺**_。

#### 正文内容

* 日本語構築手順  
	http://www.terut.net/?p=314
* windowsでDebug環境の構築（失敗っちゃた）

■gemfiles に「gem 'ruby-debug19'」を追加  
■bundle install を実行する  
■libv8 -v '3.3.10.4'　がインストールされてないです。  

	原因：Checking for Python...Unable to build libv8: Python not found!
	Pythonの環境がないです。

	ご参照
	http://stackoverflow.com/questions/9174328/fatal-error-while-bundle-install-while-installing-libv8

■ActivePythonをインストールする  
■再Bundle install して、下記のエラーができ来ました。  
	Installing therubyracer (0.9.10) with native extensions
