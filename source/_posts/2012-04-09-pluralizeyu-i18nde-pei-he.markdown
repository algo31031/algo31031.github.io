---
layout: post
title: "pluralize与i18n的配合"
date: 2012-04-09 10:56:15 +0800
comments: true
categories: rails
---
locales文件

{% codeblock lang:yml en.yml %}
:en   
  :nights     
    :one => '1 night',     
    :other => '%{count} nights' 
{% endcodeblock %}    

{% codeblock lang:yml zh-TW.yml %}    
:zh-TW   
  :nights     :one => '1 晚',     :other => '%{count} 晚' 
{% endcodeblock %}


view层

{% codeblock lang:erb %}
    view <%= t( ".nights", count: @nights ) %> 
{% endcodeblock %}