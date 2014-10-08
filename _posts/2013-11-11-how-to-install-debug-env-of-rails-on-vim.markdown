---
layout: post
title: "如何在vim中进行ruby，rails的debug(针对ruby>=1.9 && use debugger-xml gem)"
date: 2013-11-11 15:48
comments: true
tags: [rails, vim]
---

#### 概要说明
关于具体的构筑手顺，参照vim-ruby-debugger中的Readme内容即可。
这里主要是写一些自己在构筑构成中碰到一些问题和注意事项。

#### 环境
CentOS 6.2  
ruby 1.9.3p125 (2012-02-16 revision 34643)

#### 参考文章

* [Railsアプリをvimでステップ実行する方法](http://nobyu.hatenadiary.jp/entry/20111204/1322982165)
* [vim-ruby-debugger plugin source in github](https://github.com/astashov/vim-ruby-debugger)
* [debugger-xml plugin source in github](https://github.com/astashov/debugger-xml)

#### 事前准备
确认CentOS中是否有以下package。  

* libXt  
* libXt-devel  
* glib  
* gtk2  
* gtk2-devel  

特别注意：  

1. 关于glib，gtk2，gtk2-devel，在vim源代码目录的src/INSTALL文件中有说明，如果要打开clientserver开关的话，需要安装这些package。  
2. 一定要确认X11环境是否已经具备，特别是libXt-devel package。我构筑的时候就是没有这个环境，然后花费了很多时间。

<!-- more -->

#### vim从源代码安装

* 下载vim的源代码。 [vim官网](http://www.vim.org/)
* 进入vim 源代码目录，依次执行下面的命令。

```
./configure --with-features=huge --enable-multibyte --enable-pythoninterp --enable-rubyinterp --enable-cscope --enable-gui=gtk2 --prefix=$HOME/mytool --enable-gtk2-check --with-x
make
make install
```

* 由于我以前有vim的版本，并且vimfiles已经存在，所以为了让新安装的vim7.4也使用以前的vimfiles，做成了以下链接。  
`ln -s /usr/share/vim/vimfiles ~/.vim`

* 由于我把新版本安装到 `$HOME/mytool` 中，所以为了是默认的vim命令就是最新版本，在 `~/.bashrc` 中添加了如下内容。  
`export path=$HOME/mytool/bin:$PATH`

特别注意：

1. 如果没有指定--prefix，那么会默认安装vim命令到/usr/bin,如果以前已经有vim版本，那么就会覆盖掉。所以如果想保留以前版本的话，最好使用--prefix参数，或者备份以前的vim命令。  
2. 在执行完 `./configure` 后要确认一下屏幕输出的结果，主要是一些check是否都是yes，比如，以上提到的事前准备中的package不存在的话，那么会有一些头文件找不到的错误。  
3. make完成后，`src/auto/config.log` 中的信息也很有用，出错的时候可以查看这个文件中是否有一些error。

#### 关于debugger-xml的版本问题
正常来讲，vim的clientserver，ruby，signs机能打开后，装入vim-ruby-debugger插件，并且本地安装了debugger-xml gem应该就是可以debug了。  
但是，我使用了 `debugger-xml 0.3.2` 却总是无法启动 Rdebugger。大概调查了一下，可能是以下文件中，`ENV['IDE_PROCESS_DISPATCHER']`　无法取得的原因。  

[source link](https://github.com/astashov/debugger-xml/blob/master/lib/debugger/xml/multiprocess/pre_child.rb)

```ruby
# debugger-xml/lib/debugger/xml/multiprocess/pre_child.rb

acceptor_host, acceptor_port = ENV['IDE_PROCESS_DISPATCHER'].split(":")
acceptor_host, acceptor_port = '127.0.0.1', acceptor_host unless acceptor_port
```

后来把debugger-xml的版本切换回 `0.2.0` 以后就可以正常 Rdebugger 了。
