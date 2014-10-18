---
layout: post
title: "rails server是如何启动的"
date: "2014-10-17 10:14:41 +0800"
tags: [rails启动流程]
description: "分析了rails server是如何启动的，特别注意，Rails源码版本：4.1.6"
---

当我们启动一个rails应用时，比如我们运行`rails server`命令，到底后台是如何运行的？

首先确认rails命令的路径，在项目目录下执行 `which rails` 命令，结果如下：

> /Users/moyan/.rvm/gems/ruby-2.1.1/bin/rails


该rails命令文件主要内容：

```ruby
...
gem 'railties', version
load Gem.bin_path('railties', 'rails', version)
```

Gem.bin_path的执行结果：
> /Users/moyan/.rvm/gems/ruby-2.1.1/gems/railties-4.1.6/bin/rails

文件关键内容：

```ruby
...
require "rails/cli"
```

rails/cli.rb关键内容：

```ruby
require 'rails/app_rails_loader'
# If we are inside a Rails application this method performs an exec and thus
# the rest of this script is not run.
Rails::AppRailsLoader.exec_app_rails

# 这里后面还有一些内容，但是如上面的注释所说，如果在一个rails app的目录中，这些后面的脚本就不会被执行了。
```

这里的exec_app_rails就相当于执行下列命令：  
_这里的bin/rails就是rails app目录下的可执行文件_

> exec ruby bin/rails server

RailsDemoApp/bin/rails内容：

```ruby
#!/usr/bin/env ruby
begin
  load File.expand_path("../spring", __FILE__)
rescue LoadError
end
APP_PATH = File.expand_path('../../config/application',  __FILE__)
require_relative '../config/boot'
require 'rails/commands'
```

这里主要干了以下事情：

* 定义了APP_PATH
* require config/boot文件，这个文件主要是进行bundle/setup，也就是检查Gemfile的内容
* require 'rails/commands'

rails/commands.rb中关键代码：

```ruby
Rails::CommandsTasks.new(ARGV).run_command!(command)
```

run_command!方法代码：

```ruby
def run_command!(command)
  command = parse_command(command)
  if COMMAND_WHITELIST.include?(command)
    send(command)
  else
    write_error_message(command)
  end
end
```

其中调用了send(command),因为我们执行的rails server，就相当于这里又调用了server方法：

```ruby
def server
  set_application_directory!
  require_command!("server")

  Rails::Server.new.tap do |server|
    # We need to require application after the server sets environment,
    # otherwise the --environment option given to the server won't propagate.
    require APP_PATH
    Dir.chdir(Rails.application.root)
    server.start
  end
end
```

这里有以下关键的地方：

* 这里require了APP_PATH，这个PATH就是该应用目录下的config/application.rb
* 调用了server.start方法，就相当于正式启动了server

#### 结语

这里只是简单的分析了server的启动流程，其实针对一个Rails application，它的启动过程中还做了更多的事情。
有时间的话，再继续写一些application的启动流程相关的内容。

#### 追加信息
[2014-10-18] -- 
接下来的分析参照：
[Rails Application启动流程]({% post_url 2014-10-18-rails-application-booting-process %})
