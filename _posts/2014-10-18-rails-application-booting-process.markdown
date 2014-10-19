---
layout: post
title: "Rails Application启动流程"
date: "2014-10-18 20:28:49 +0800"
description: "分析了Rails应用的启动流程，特别注意，Rails源码版本：4.1.6。这里是自己看Rails源码的一个思路整理，不一定适合作为一个教程。"
comments: true
tags: [rails启动流程]
---

在之前的博文
[rails server是如何启动的]({% post_url 2014-10-17-how-rails-server-command-work %})
中分析了rails server这个命令最终是如何启动的，本篇接着分析一个Rails Application到底是如何被构建成功的。

#### 事前说明

如果要分析Rails的启动流程的话，需要对以下知识有一些了解：

* [rack](https://github.com/rack/rack)
* middleware（可以参考博文：
[Rails应用中的middleware们]({% post_url 2014-10-19-middleware-in-rails-application %})
）

以下是一些小tips，可以事前了解：

* Applicaiton, Engine, Railtie的继承关系：Rails::Application < Rails::Engine < Rails::Railtie
* 相对应的Configuration类的继承关系：Rails::Application::Configuration < Rails::Engine::Configuration < Rails::Railtie::Configuration
* Rails::Railtie::Configuration 中的变量都是类变量
* Rails::Engine::Configuration 和 Rails::Application::Configuration 中的变量都是实例变量

<!-- more --> 

#### Rails Application API Guide

在Rails的
[Api文档](http://api.rubyonrails.org/)
中(Rails 4.1.6)，关于Application这个类的介绍中写到了一个Rails Application的启动流程，原文如下：

```
 The application is also responsible for setting up and executing the booting process. From the moment you require “config/application.rb” in your app, the booting process goes like this:

1) require "config/boot.rb" to setup load paths
2) require railties and engines
3) Define Rails.application as "class MyApp::Application < Rails::Application"
4) Run config.before_configuration callbacks
5) Load config/environments/ENV.rb
6) Run config.before_initialize callbacks
7) Run Railtie#initializer defined by railties, engines and application.
   One by one, each engine sets up its load paths, routes and runs its config/initializers/* files.
8) Custom Railtie#initializers added by railties, engines and applications are executed
9) Build the middleware stack and run to_prepare callbacks
10) Run config.before_eager_load and eager_load! if eager_load is true
11) Run config.after_initialize callbacks
```

接下来我们就按照这个流程，分别看看具体在Rails源码中是如何进行的。

#### Server启动

参照博文
[rails server是如何启动的]({% post_url 2014-10-17-how-rails-server-command-work %})
中最后server启动部分的代码，关键的部分如下：

```ruby
def server
  ......
  Rails::Server.new.tap do |server|
    # 关键点一
    require APP_PATH
    Dir.chdir(Rails.application.root)
    # 关键点二
    server.start
  end
end
```

分为下面两个关键部分：

* require APP_PATH
* server.start

#### require APP_PATH

这里的APP_PATH，是定义在rails应用的bin/rails文件中，其实就是`config/application.rb`文件。

在require该文件时，发生了很多事情，对应API文档中的**流程1)～4)**。

##### 步骤1: require "config/boot.rb" to setup load paths

这个是在application.rb文件的一开始就被require的。

##### 步骤2: require railties and engines

这个是在boot.rb文件中，会进行bundle/setup，就相当于把Gemfile中使用的一些Gem（这些Gem中包含各种Engine或者Railtie），都加载到load path中。

##### 步骤3: Define Rails.application as "class MyApp::Application < Rails::Application"

Rails::Application被继承的时候，在会执行以下代码：

```ruby
# rails/railties/lib/rails/application.rb
def inherited(base)
  # 这里的super是指engine
  super
  base.instance
end
```

这里会调用base.instance，这里的base其实就是自己的Application类（MyApp::Application），而instance方法是使用了一个单例模式，在iinstance方法中就是调用了new并保存在@instance中。

```ruby
# rails/railties/lib/rails/railtie.rb
def instance
  @instance ||= new
end
```

而在Rails::Application的initialize方法中，主要做了以下事情：

* 设置了Rails.application
* 加载lib目录到load path
* 执行了config.before_configuration callbacks(**步骤4**)

代码参照如下：

```ruby
def initialize(initial_variable_values = {}, &block)
  super()
  @initialized       = false
  @reloaders         = []
  @routes_reloader   = nil
  @app_env_config    = nil
  @ordered_railties  = nil
  @railties          = nil
  @message_verifiers = {}

  Rails.application ||= self

  # 加载lib目录到load path
  add_lib_to_load_path!

  # 4)Run config.before_configuration callbacks
  # 因为这个时候还没有调用config方法，也就是还没有进行config配置
  ActiveSupport.run_load_hooks(:before_configuration, self)

  initial_variable_values.each do |variable_name, value|
    if INITIAL_VARIABLES.include?(variable_name)
      instance_variable_set("@#{variable_name}", value)
    end
  end

  instance_eval(&block) if block_given?
end
```

#### server.start

server方法其实会根据参数等来选择一个符合rack规范的具体的server，比如thin，webrick等。具体关于server的部分不做详细介绍，主要介绍start的部分。

```ruby
# rails/railties/lib/rails/commands/server.rb
def start
  # 打印启动信息
  print_boot_information
  trap(:INT) { exit }
  # 创建tmp目录
  create_tmp_directories
  # 是否打印log到标准输出
  log_to_stdout if options[:log_stdout]

  # 调用rack/server的start方法
  super
ensure
  # The '-h' option calls exit before @options is set.
  # If we call 'options' with it unset, we get double help banners.
  puts 'Exiting' unless @options && options[:daemonize]
end
```

Rails::Server是继承自Rack::Server的，所以start方法最后的super是调用Rack::Server中的start方法：

```ruby
# rack/lib/rack/server.rb
def start &blk
  ......

  check_pid! if options[:pid]

  # Touch the wrapped app, so that the config.ru is loaded before
  # daemonization (i.e. before chdir, etc).
  # 关键点
  wrapped_app

  daemonize_app if options[:daemonize]

  write_pid if options[:pid]

  trap(:INT) do
    if server.respond_to?(:shutdown)
      server.shutdown
    else
      exit
    end
  end

  # 关键点
  server.run wrapped_app, options, &blk
end
```

这里的关键点就是这个`wrapped_app`方法，可以看到最后server run的也就是这个warpped_app。

```ruby
# rack/lib/rack/server.rb
def wrapped_app
  @wrapped_app ||= build_app app
end
```

其中的`app`就是执行config.ru，最终会得到一个app。而build_app的作用是为了在得到的app的上层再包装上server的middleware。

```ruby
# rack/lib/rack/builder.rb
def self.new_from_string(builder_script, file="(rackup)")
  # 执行了config.ru文件, 然后调用to_app
  eval "Rack::Builder.new {\n" + builder_script + "\n}.to_app",
    TOPLEVEL_BINDING, file, 0
end
```

在执行config.ru文件的过程中，又涉及到了rails启动过程中的**步骤5～11**。

config.ru文件内容：

```ruby
# RailsDemoApp/config.ru
# This file is used by Rack-based servers to start the application.
require ::File.expand_path('../config/environment',  __FILE__)
run Rails.application
```

在environment.rb文件中，调用了`Rails.application.initialize!`，接下来的步骤都是在该方法中完成的。

```ruby
# RailsDemoApp/config/environment.rb
# Load the Rails application.
require File.expand_path('../application', __FILE__)

# Initialize the Rails application.
Rails.application.initialize!
```

Rails.application.initialize!代码如下：

```ruby
# rails/railties/lib/rails/application.rb
# Initialize the application passing the given group. By default, the
# group is :default
def initialize!(group=:default) #:nodoc:
  raise "Application has been already initialized." if @initialized
  # 执行了所有的注册的initializers
  run_initializers(group, self)
  @initialized = true
  self
end
```

在上面代码中看到，执行了所有的注册的initializers。
在Rails中的initializer分为以下三类：

* Bootstrap的initializer
* Application, Engine，Railtie注册的initializer
* Finisher的initializer

从以下方法源码可以看出：

```ruby
# rails/railties/lib/rails/application.rb
# 注意这里重写了父类的该方法initializers
def initializers #:nodoc:
  Bootstrap.initializers_for(self) +
  # 这里的super得到的应该是my application的所有initializers,但是这些initializers被定义在railtie，engine，application里面，因为继承关系, 最终得到的应该是  每个gem的railtie的initializers + 每个gem的engine的initializers + my appilcation的initializers, 可以通过在rails console 中，Rails.application.send(:initializers)的结果来证实
  railties_initializers(super) +
  Finisher.initializers_for(self)
end
```

那么启动流程中的步骤5～11都是一些特定的initializer来执行的。

##### 步骤5: Load config/environments/ENV.rb

```ruby
# rails/railties/lib/rails/engine.rb
# 这个就是启动流程中的 5) Load config/environments/ENV.rb, 特别注意这里用了before
initializer :load_environment_config, before: :load_environment_hook, group: :all do
  paths["config/environments"].existent.each do |environment|
    require environment
  end
end
```

##### 步骤6: Run config.before_initialize callbacks

```ruby
# rails/railties/lib/rails/application/bootstrap.rb
initializer :bootstrap_hook, group: :all do |app|
  ActiveSupport.run_load_hooks(:before_initialize, app)
end
```

##### Rails应用config目录下的initializers执行的源码位置

```ruby
# rails/railties/lib/rails/engine.rb
# 这里就是config目录下的initializers执行的位置
initializer :load_config_initializers do
  config.paths["config/initializers"].existent.sort.each do |initializer|
    load_config_initializer(initializer)
  end
end
```

##### 步骤9: Build the middleware stack and run to_prepare callbacks

build\_middleware_stack，该方法alias到app方法，这个方法是个重要的方法，这里就是把所有middleware都一层一层嵌套然后得到最终的app。

```ruby
# rails/railties/lib/rails/application/finisher.rb
initializer :build_middleware_stack do
  build_middleware_stack
end

# This needs to happen before eager load so it happens
# in exactly the same point regardless of config.cache_classes
# 9) run to_prepare callbacks
initializer :run_prepare_callbacks do
  ActionDispatch::Reloader.prepare!
end
```

##### 步骤10: Run config.before\_eager\_load and eager\_load! if eager_load is true

```ruby
# rails/railties/lib/rails/application/finisher.rb
initializer :eager_load! do
  if config.eager_load
    ActiveSupport.run_load_hooks(:before_eager_load, self)
    config.eager_load_namespaces.each(&:eager_load!)
  end
end
```

##### 步骤11: Run config.after_initialize callbacks

```ruby
# rails/railties/lib/rails/application/finisher.rb
# All initialization is done, including eager loading in production
initializer :finisher_hook do
  ActiveSupport.run_load_hooks(:after_initialize, self)
end
```

至此，Rails App的启动流程就完成了。Enjoy it~

#### 其他

* eager load: 预加载
* lazy load: 延迟加载
* ancestors(Module) : 返回所有include的mod数组，ruby API

#### 参考链接

* [alias, alias\_method, alias\_method\_chain区别](http://blackanger.blog.51cto.com/140924/355102/)
* [讲述rails的lazy load hooks](http://simonecarletti.com/blog/2011/04/understanding-ruby-and-rails-lazy-load-hooks/)
