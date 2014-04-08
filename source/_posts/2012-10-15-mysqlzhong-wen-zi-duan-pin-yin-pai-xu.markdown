---
layout: post
title: "MySQL中文字段拼音排序"
date: 2012-10-15 09:47:43 +0800
comments: true
categories: MySQL
---
不想改变表定义及默认编码的情况，将字段先转换成gbk编码再排序：

    SELECT * FROM table ORDER BY CONVERT( chinese_field USING gbk ) ;

前提是在安装mysql时安装了gbk字符集，不然会报错：

    #1115 - Unknown character set: 'gbk'

在编译源码时加上gbk编码即可，如果已经安装好了，重新编译再安装，重新编译安装一般不会影响mysql的已有设置，包括数据都不会受到影响。