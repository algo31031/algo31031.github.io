---
layout: post
title: "database.yml里password如果全部为数字需要加''"
date: 2012-03-26 17:27:06 15:59:01 +0800
comments: true
categories: rails
---
database.yml里面密码采用全数字时，加 ' ', 否则运行rake时报错： can't convert Fixnum into String  