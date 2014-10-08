---
layout: post
title: "LinuxでNetBeansでrailsのDebug環境の構築"
date: 2013-01-25 14:58
comments: true
tags: [linux, rails, env install]
---

#### 前言

这里只是单纯的记录了自己在ubuntu中为NetBeans构筑Debug环境过程中的一些
注意事项，主要是碰到一些错误。  
特别注意: _**这里不是手顺**_。

#### 正文内容

■一番最新のruby-debug-base19 (0.11.26)のインストールが必要です。
    ruby-debug-base19 (0.11.26)
	ruby-debug-ide19 (0.4.12)
	ruby-debug19 (0.11.6)

	gem install ruby-debug19 -- --with-ruby-include=C:\ruby\Ruby193\include\ruby-1.9.1\ruby-1.9.3-p286
	curl -OL http://rubyforge.org/frs/download.php/75415/ruby-debug-base19-0.11.26.gem
	curl -OL http://rubyforge.org/frs/download.php/75414/linecache19-0.5.13.gem

	====ご参照===
	https://gist.github.com/1457544
	http://ruby-china.org/topics/843

■environment.rbに下記のメソッドの追加が必要です。
	# add by dairg 20121101
	class String
	  def is_binary_data?
		( self.count( "^ -~", "^\r\n" ).fdiv(self.size) > 0.3 || self.index( "\x00" ) ) unless empty?
	  end
	end

	=====ご参照=====
	http://stackoverflow.com/questions/8961367/aptana-3-ruby-debugger-exception-in-debugthread-loop-undefined-method-is-bin/9043481#9043481

■linecache19 (0.5.13)
