---
layout: post
title: "在Rails中使用ElasticSearch进行检索"
date: "2015-05-19 08:40:46"
tags: [rails, elasticsearch]
---

#### 概要说明
这篇博客介绍自己对ElasticSearch(下面简称ES)的入门使用过程。  
ES底层的搜索实现是基于lucene的。

#### 安装ElasticSearch
Mac系统的话，推荐使用HomeBrew来安装各种软件。

```
brew install elasticsearch
```

如果是Linux发行版的话，可以通过apt-get,yum进行安装。
参考官网的[下载页面的安装说明](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-repositories.html)。

以Ubuntu为例：

```bash
# 下载安装public signing key
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
# 添加Repo到sources.list
echo "deb http://packages.elastic.co/elasticsearch/1.5/debian stable main" | sudo tee -a /etc/apt/sources.list
# 更新repo并且安装ES
sudo apt-get update && sudo apt-get install elasticsearch
# 如果要设置为开机自启动，执行下面命令
sudo update-rc.d elasticsearch defaults 95 10
```

安装完成的话，访问http://localhost:9200，如果出现一下结果就是OK了。

```json
{
  "status" : 200,
  "name" : "Earth Lord",
  "cluster_name" : "elasticsearch_moyan",
  "version" : {
    "number" : "1.4.4",
    "build_hash" : "c88f77ffc81301dfa9dfd81ca2232f09588bd512",
    "build_timestamp" : "2015-02-19T13:05:36Z",
    "build_snapshot" : false,
    "lucene_version" : "4.10.3"
  },
  "tagline" : "You Know, for Search"
}
```

<!-- more -->

#### 安装中文分词器ik

ES默认的中文分词器不是太好，比如针对`中国`这个词，会分成`中`和`国`两个词来搜索，而我们期待的可能是作为一个整体的词进行搜索。
所以需要安装中文分词。

这里介绍安装的是有名的中文分词器：[elasticsearch-analysis-ik](https://github.com/medcl/elasticsearch-analysis-ik)

安装的话，参考项目主页的Readme就可以。但是需要注意的是，ik的版本和es的版本会有对应关系。所以安装的时候要先确认自己的es
的版本，然后安装对应的ik的版本。关于对应关系，在项目主页的Readme中已经罗列出来了。

我是通过参照Readme最后的介绍，重新编译安装的。步骤如下:

```
# clone项目
git clone https://github.com/medcl/elasticsearch-analysis-ik
cd elasticsearch-analysis-ik
# 编译
mvn compile
# 打包
mvn package
# 安装ik分词器插件,这里的plugin命令就是es提供的用来对es插件进行管理的
plugin —install analysis-ik —url file:///#{project_path}/elasticsearch-analysis-ik/target/releases/elasticsearch-analysis-ik-1.3.0.zip
```

安装完成后，还需要拷贝分词文件。也就是把项目`config/`目录下的`ik`文件夹拷贝到es的config目录下。

最后在es的配置文件`elasticsearch.yml`中，添加ik相关配置：

```
index:
  analysis:                   
    analyzer:      
      ik:
          alias: [ik_analyzer]
          type: org.elasticsearch.index.analysis.IkAnalyzerProvider
      ik_max_word:
          type: ik
          use_smart: false
      ik_smart:
          type: ik
          use_smart: true
```

安装完成后，重启ES server，然后在浏览器中访问`http://localhost:9200/articles/_analyze?analyzer=ik&text=中文分词&pretty`，
如果结果如下所示，证明中文分词OK。  
(这里的articles是创建的索引，如果不存在，替换为已经存在的索引，如果没有，可以新建一个索引进行测试)

```json
{
  "tokens" : [ {
    "token" : "中文",
    "start_offset" : 0,
    "end_offset" : 2,
    "type" : "CN_WORD",
    "position" : 1
  }, {
    "token" : "分词",
    "start_offset" : 2,
    "end_offset" : 4,
    "type" : "CN_WORD",
    "position" : 2
  } ]
}
```

#### Rails工程

[这里(github)](https://github.com/MakeItEasy/esdemo)是我做的一个ES的demo工程，实现了简单的检索。

如果是创建新的工程的话，推荐使用[这个连接](https://github.com/elastic/elasticsearch-rails/tree/master/elasticsearch-rails#rails-application-templates)中介绍的从template来构建的方式。

```ruby
rails new searchapp --skip --skip-bundle --template https://raw.github.com/elasticsearch/elasticsearch-rails/master/elasticsearch-rails/lib/rails/templates/01-basic.rb

rails new searchapp --skip --skip-bundle --template https://raw.github.com/elasticsearch/elasticsearch-rails/master/elasticsearch-rails/lib/rails/templates/02-pretty.rb

rails new searchapp --skip --skip-bundle --template https://raw.github.com/elasticsearch/elasticsearch-rails/master/elasticsearch-rails/lib/rails/templates/03-expert.rb
```

要注意的是：如果想通过02或者03模版来创建工程的话，必须是从01开始执行，而不是只执行指定的模版命令。

#### 注意点

* 如果导入es时，Rails工程的数据库中已经有现存的数据，那么需要对现有的数据进行索引，可以通过以下命令进行：

```
rake environment elasticsearch:import:model CLASS='Article' BATCH=100 FORCE=y
```

执行这个task的前提是，在`lib/tasks/`目录下有文件`elasticsearch.rake`, 内容为:

```ruby
require 'elasticsearch/rails/tasks/import'
```

* 如果对某个字段要是用上文介绍的ik分词器，那么在model文件中需要进行mapping配置，例子如下：

```ruby
settings index: { number_of_shards: 3 } do
  mappings do
    indexes :title, type: 'string', analyzer: 'ik'
    indexes :keywords, type: 'string', analyzer: 'ik'
    indexes :content, type: 'string', analyzer: 'ik'
  end
end
```

* 如果希望model进行增删改操作后，索引也能同步更新，那么在model文件中需要引入callbacks，如下：

```ruby
include Elasticsearch::Model
include Elasticsearch::Model::Callbacks
```

#### 参考链接
* [ elasticsearch官网 ](https://www.elastic.co/)
* [elasticsearch中文向导](http://www.elasticsearch.cn/)
* [happycast视频](http://haoduoshipin.com/v/104)


(The Eed)
