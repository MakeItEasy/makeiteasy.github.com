---
layout: post
title: "关于Rails中的SSL"
date: 2013-05-22 10:59
comments: true
categories: [rails, ssl]
---

#### 关于force SSL请求的配置
只需要在config文件（比如application.rb）中添加如下配置:  
`config.force_ssl = true`  

关于如何启动支持SSL的server，可以使用thin服务器。具体做法如下:  

* 在Gemfile中添加如下内容:  

```ruby
group :development do
  gem 'thin'
end
```


* 启动ssl服务器的命令: `thin start --ssl`  

#### 如果部分请求force ssl, 部分请求还是http的情况

* 在配置文件中做如下配置  

```ruby
config.middleware.insert_before ActionDispatch::Static, Rack::SSL, :exclude => proc { |env| env['HTTPS'] != 'on' }

# 注1：可能需要在头部 require 'rack/ssl'
# 注2：exclude就是针对那些请求不会包装成https的请求。比如上面的配置意思就是说，  
#     如果客户请求的是http（比如在地址栏输入的是http://localhost），那么就去请求http
```

* 在需要强制https的controller中添加 `before_filter`, 例如：  

```ruby
before_filter :force_ssl

def force_ssl
  if !request.ssl?
	redirect_to :protocol => 'https', :port => 3001
  end
end
```

* 这个时候测试的话可能需要启动两个服务器  

```
thin start -p 3000  
thin start --ssl -p 3001  
```


