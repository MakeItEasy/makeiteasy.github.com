---
layout: post
title: "windows下安装ctags"
date: 2013-03-07 10:21
comments: true
categories: [vim]
---

#### 背景
在windows中安装了gvim后，如果使用了taglist等插件后，启动gvim，
会报错，因为找不到ctags命令，而在linux下，默认会安装ctags命令。

#### 安装过程

1. 下载安装包: [http://ctags.sourceforge.net/](http://ctags.sourceforge.net/)
2. 将下载的zip包解压到 `vim/vim73/` 目录下
3. 将 `ctags/` 目录添加到环境变量path中
