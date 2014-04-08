---
layout: post
title: "undefined method 'rewrite' for #<String:0xaf85690>的解决"
date: 2011-01-05 12:56:43 +0800
comments: true
categories: rails
---
简而言之，就是代码中在有使用`link_to`的时候 ，出现了`@url`

***

[点此查看原文](http://www.concept47.com/austin_web_developer_blog/ruby-on-rails/ruby-on-rails-gotcha-undefined-method-rewrite-for/)

If you get this error, and the error message is pointing you to a “link_to” call or something similar, then you may be using an instance variable that’s called ‘@url’ too.

This blog post did talk about the problem but it seemed limited to models only. I finally discovered that, in my case, I was using ‘@url’ in the controller for the view where I was making the ‘link_to’ call.

Long story short, if you see this error, comb through your code (models, controllers and views) for any variables that are called ‘@url’  and change them.