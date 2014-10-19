---
layout: post
title: "Rails应用中的middleware们"
date: "2014-10-19 10:17:38 +0800"
comments: true
tags: [rails]
---

[Rails应用中的middleware们]({% post_url 2014-10-19-middleware-in-rails-application %})

middleware在rack规范中是一个很重要的概念。在rails中，一个app，其实就是各种middleware一层一层嵌套起来工作的。

在rails app中，middleware可以分为三类：

* Server的middleware
* 在config.ru中use的middleware
* 在application.config中use的middleware

他们的嵌套顺序也是按照上面的顺序，即server middleware 包含 config.ru middleware 包含 application middleware

#### 前提说明

本文中涉及到的rails源码针对的版本是：**4.1.6**

#### Server中的middleware

在Rack::Server中有以下代码，这里的build_app其实就是为了给后面得到的app包装上server的middleware的。

```ruby
# rack/lib/rack/server.rb
def wrapped_app
  @wrapped_app ||= build_app app
end
```

比如Rails::Server中重写父类Rack::Server的middleware方法：

```ruby
#
def middleware
  middlewares = []
  middlewares << [Rails::Rack::Debugger] if options[:debugger]
  middlewares << [::Rack::ContentLength]

  # FIXME: add Rack::Lock in the case people are using webrick.
  # This is to remain backwards compatible for those who are
  # running webrick in production. We should consider removing this
  # in development.
  if server.name == 'Rack::Handler::WEBrick'
    middlewares << [::Rack::Lock]
  end

  Hash.new(middlewares)
end
```

#### config.ru中的middleware

默认Rails生成的新应用的config.ru文件内容如下：

```ruby
# This file is used by Rack-based servers to start the application.
require ::File.expand_path('../config/environment',  __FILE__)
run Rails.application
```

其实在`run Rails.application`之前，是可以use一些middleware的。比如：

```ruby
# This file is used by Rack-based servers to start the application.
require ::File.expand_path('../config/environment',  __FILE__)
use Rack::ShowExceptions
use Rack::ETag
use OTHER Middleware
......
run Rails.application
```

那么这里use的middleware，就被包装在了Rails app的middleware之前。

#### Application中的middleware

在rails应用中可以通过config的如下方法来编辑application的middleware：  
参考：
<http://guides.rubyonrails.org/configuring.html#configuring-middleware>

* config.middleware.use Magical::Unicorns
* config.middleware.insert_before ActionDispatch::Head, Magical::Unicorns
* config.middleware.insert_after ActionDispatch::Head, Magical::Unicorns
* config.middleware.swap ActionController::Failsafe, Lifo::Failsafe
* config.middleware.delete "Rack::MethodOverride"

#### rake middleware命令

在rails应用目录中执行`rake middleware`命令，会打印app的middlewares。但是特别注意的是，只会打印上面提到的application中的middleware，而server中，以及config.ru中use的middleware是不会被打印出来的。

从以下middleware.rake的源码可以看出这一点：

```ruby
# rails/railties/lib/rails/tasks/middleware.rake
desc 'Prints out your Rack middleware stack'
task middleware: :environment do
  Rails.configuration.middleware.each do |middleware|
    puts "use #{middleware.inspect}"
  end
  puts "run #{Rails.application.class.name}.routes"
end
```
