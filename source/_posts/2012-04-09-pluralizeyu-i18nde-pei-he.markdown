---
layout: post
title: "pluralize与i18n的配合"
date: 2012-04-09 10:56:15 +0800
comments: true
categories: rails
---
[原文出处](http://ruby-taiwan.org/topics/142)

locales文件

    en.yml :en   :nights     :one => '1 night',     :other => '%{count} nights' 
    zh-TW.yml :zh-TW   :nights     :one => '1 晚',     :other => '%{count} 晚' 
    
view层

    view <%= t( ".nights", count: @nights ) %> 