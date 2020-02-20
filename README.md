### 简介

This is my blog in github.

### 如何在新环境配置自己的博客环境

1. gem install jekyll
2. clone this repo
3. cd repo directory & bundle install
4. jekyll serve --watch


### 添加Slide的支持

```
git submodule add https://github.com/hakimel/reveal.js.git
```

由于添加了`submodule`,所以在新环境clone后，需要执行以下命令才能保证文件完整。

```
git submodule update --init --recursive
```

### 20200219 在windows环境下构建环境

#### 安装Ruby环境

本来一开始打算安装RVM来进行Ruby环境管理，RVM不直接支持windows，但是可以使用cygwin等环境，然后安装cygwin后，`rvm install 2.7` 的时候，没有针对cygwin的现成的ruby binary包，所以需要手动从源码安装。所有没有使用rvm。

windows安装ruby，从如下链接下载即可：

https://rubyinstaller.org/downloads/

> 特别注意：不要追求新版本，我一开始下载了ruby2.7，后来很多Gem要求的ruby环境都小于2.7。所以后来切换到2.6版本。

#### 配置Jekyll环境

按照如下脚本执行：

```
cd makeiteasy.github.com
gem install bundle
bundle install
```

在执行 `bundle install` 的过程中，发生了gem依赖ruby环境版本的问题，所以我更新了 `github-pages` gem。

```
bundle update github-pages
bundle install
```

执行完成之后就可以使用jekyll了。

```
# 启动jekyll
jekyll serve -w
```