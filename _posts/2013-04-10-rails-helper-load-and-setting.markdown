---
layout: post
title: "rails中helper的加载和相关设置，以及如何设置不全部加载所有helper"
date: 2013-04-10 10:30
comments: true
categories: [rails]
---

#### 关于helper的总结

* rails默认为controller加载所有的helper，所以helper 方法无论写在那个文件中，都可以在view层调用。  
* 设置不要默认加载所有的helper

```ruby
# Rails >= 3.1 时，通过在配置文件中，添加如下设置:  
config.action_controller.include_all_helpers = false

# Rails < 3.1,默认在ApplicationController中有以下行，删除即可  
helper :all
```

* 如果经过以上的配置，那么controller默认就只会加载自己对应的helper，如果controller有继承关系，那么也会加载父类对应的helper  
* 通过在controller中使用以下方法，可以加载制定的helper: `helper CommonHelper`  

