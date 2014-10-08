---
layout: post
title: "在局域网（LAN）中访问webrick服务器速度过慢"
date: 2013-01-25 16:09
comments: true
tags: [rails]
---

#### 问题现象描述
在Linux虚拟机中进行rails开发和server构建，通过外边的windows访问应用时，
速度很慢。

#### 对策一
参考帖子:  [http://www.flatws.cn/article/program/ruby/2011-05-01/23456.html](http://www.flatws.cn/article/program/ruby/2011-05-01/23456.html)

帖子内容总结（修改了两个文件）  

修改文件1

	路径：/usr/local/rvm/rubies/ruby-1.9.3-p286/lib/ruby/1.9.1/webrick
	文件：server.rb
	方法：GenericServer#start_thread
	代码：
	addr = sock.peeraddr 改为 addr = sock.peeraddr(:numeric)

修改文件2

	路径：/usr/local/rvm/rubies/ruby-1.9.3-p286/lib/ruby/1.9.1/webrick
	文件：httprequest.rb
	方法：HTTPRequest#parse
	代码：
	@peeraddr = socket.respond_to?(:peeraddr) ? socket.peeraddr : []
	改为
	@peeraddr = socket.respond_to?(:peeraddr) ? socket.peeraddr(:numeric ) : []
	
#### 对策二
修改内容如下

	修改文件：#{RUBY_HOME}/lib/ruby/1.9.1/webrick/config.rb 
	:DoNotReverseLookup => nil
	改为
	:DoNotReverseLookup => true
