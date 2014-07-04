---
layout: post
title: "apache下设置AddDefaultCharset的问题"
date: 2014-07-04 14:09:29 +0800
comments: true
categories: [deploy, apache]
---
通过iframe在子页面里加载scorm课件, utf-8编码的可正常显示, shift_jis的乱码  
可是子页面里已经通过meta设置了charset

怀疑是被其他地方的charset覆盖掉, 后来找到篇文章说可能有apache的AddDefaultCharset设置有关

本地测试验证了怀疑:  
>
  不加AddDefaultCharset            正常显示  
  加上AddDefaultCharset utf-8      乱码  
  改成AddDefaultCharset shift_jis  恢复正常   

