---
layout: post
title: "rails应用中timezones的正确用法"
date: 2013-03-08 11:08
comments: true
tags: [rails]
---

#### 说明
本文参照翻译了以下博文:

[The Exhaustive Guide to Rails Time Zones](http://danilenko.org/2012/7/6/rails_timezones/)  
作者: Alexander 

#### 正文

* 得到当前时间  

> `正确做法`  
> Time.zone.now
>
> `可接受做法`  
> Time.now.in_time_zone  
> DateTime.now.in_time_zone
> 
> `错误做法`  
> Time.now  
> DateTime.now
> 

* 得到日期（比如今天，昨天等）  

> `正确做法`  
> Time.zone.today  
> Time.zone.today - 1.day
>
> `可接受做法`  
> Date.current  
> Date.yesterday
> 
> `错误做法`  
> Date.today
> 

* 生成Time（Build Time）  

> `正确做法`  
> Time.zone.local(2013, 3, 8, 10, 00)
> 
> `错误做法`  
> Time.new(2013, 3, 8, 10, 00)  
> DateTime.new(2013, 3, 8, 10, 00)
> 

* 从时间戳生成Time（Time from Timestamp）  

> `正确做法`  
> Time.zone.at(Timestamp)
> 
> `可接受做法`  
> Time.at(Timestamp).in_time_zone
> 
> `错误做法`  
> Time.at(Timestamp)
> 

* 解析时间（简单）  

> `正确做法`  
> Time.zone.parse(str)
> 
> `错误做法`  
> Time.parse(str)
> 

* 解析时间（按照严格格式）  

> `正确做法`  
> Time.zone.strptime(str, "%Y-%m-%d %H:%M %Z")  
> 注意：这种比较好的做法，需要引入作者自己写的gem。[TimeZoneExt](https://github.com/doz/time_zone_ext)
> 
> 
> `可接受做法`  
> DateTime.strptime(str, "%Y-%m-%d %H:%M %Z").in_time_zone
> 
> `错误做法`  
> DateTime.strptime(str, "%Y-%m-%d %H:%M %Z")
> 

* 日期转时间（Get Time From Date）  

> `正确做法`  
> date.beginning_of_day
> 
> `可接受做法`  
> date.to_time_in_current_zone
> 
> `错误做法`  
> date.to_time
> 

#### 注意事项
* ruby的标准库中，比如Time和DateTime类是没有考虑时区（TimeZone）的概念在里面的。
