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


