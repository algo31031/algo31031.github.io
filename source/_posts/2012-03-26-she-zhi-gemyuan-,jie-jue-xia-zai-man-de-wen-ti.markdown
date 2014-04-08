---
layout: post
title: "设置gem源，解决下载慢的问题"
date: 2012-03-26 15:10:18 16:03:03 +0800
comments: true
categories: rails
---
###问题解决的最好方法方法

使用google的DNS 8.8.8.8 / 8.8.4.4

###另一种解决方式

修改rubygems的source源

删除原有gem source 

    $ gem source -r http://rubygems.org/ 
    $ gem source -r http://production.s3.rubygems.org/ 

增加新source源 

    $ gem source -a http://production.s3.ru

###如果使用Bundler
  
修改你的 Gemfile 将 `http://rubygems.org` 改为 `http://ruby.taobao.org/`    