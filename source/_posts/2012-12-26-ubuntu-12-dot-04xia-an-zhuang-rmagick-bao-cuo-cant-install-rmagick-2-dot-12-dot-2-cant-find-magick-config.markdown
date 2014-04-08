---
layout: post
title: "Ubuntu 12.04下安装 RMagick 报错Can't install RMagick 2.12.2. Can't find Magick-config"
date: 2012-12-26 13:28:41 +0800
comments: true
categories: ubuntu
---
{% codeblock lang:bash 报错信息 %}
checking for Magick-config... no
Can't install RMagick 2.12.2. Can't find Magick-config in /home/hanbing/.rvm/gems/ruby-1.9.3-p125/bin:/home/hanbing/.rvm/gems/ruby-1.9.3-p125@global/bin:/home/hanbing/.rvm/rubies/ruby-1.9.3-p125/bin:/home/hanbing/.rvm/bin::/opt/android-sdk-linux/tools:/opt/android-sdk-linux/platform-tools:/usr/lib/lightdm/lightdm:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/home/hanbing/.rvm/bin
{% endcodeblock %}


ImageMagick正确安装后还需要安装相应的dev包： 

{% codeblock lang:bash %}
$ sudo apt-get install libmagickwand-dev 
{% endcodeblock %}    