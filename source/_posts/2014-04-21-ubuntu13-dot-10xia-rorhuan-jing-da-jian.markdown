---
layout: post
title: "ubuntu下ROR环境搭建"
date: 2013-10-21 14:09:24 +0800
comments: true
categories: [rails, ubuntu]
---
13.10/13.04下应该都可以

13.10可以先安装新立得软件软件包管理器 (13.04之后版本不默认自带这个了, 还挺有用的)
    $ sudo apt-get install synaptic  

### 安装rvm
先安装需要的工具
    $ sudo apt-get --no-install-recommends install bash curl git patch bzip2 vim

安装rvm, 关于rvm的具体信息可以[在这里看](https://rvm.io/)
    $ curl -L https://get.rvm.io | bash -s stable
    $ source /home/hanbing/.rvm/scripts/rvm

此时运行`rvm -v`应该能看到版本信息

修改`.bashrc`文件
把以下内容加到最下面一行，保存，退出
    [[ -s "/home/hanbing/.rvm/scripts/rvm" ]] && source "/home/hanbing/.rvm/scripts/rvm"  # This loads RVM into a shell session.
打开终端，找到“编辑”，“配置文件首选项”，打开“标题和命令”选项卡，把“以登录shell方式运行命令”选中

最后记得在`~/.gemrc`下添加下面一行
    gem: --no-ri --no-rdoc
这会在每次安装Gem时忽略ri和rdoc, 不然既浪费大量时间又消耗硬盘空间
  
  
### 通过rvm安装 ruby , rails
同样，先安装需要的工具
    $ rvm requirements  
这会检查你当前环境下还缺少那些rvm需要的包, 然后全部安装
  
安装Ruby 1.9.3
    $ rvm install 1.9.3 

使用ruby 1.9.3,并设置为默认的Ruby
    $ rvm use 1.9.3 --default

安装Rails
    $ gem install rails -v 3.2.13
若安装成功，运行`ruby -v`与`rails -v`,能看到相应版本信息

### 安装mysql
    $ sudo apt-get install mysql-server mysql-common mysql-client libmysqlclient-dev
  安装过程中需要自定义`root`用户的密码并确认

### 安装Sublime Text2
    $ sudo add-apt-repository ppa:webupd8team/sublime-text-2  
    $ sudo apt-get update  
    $ sudo apt-get install sublime-text

[下载安装Package Control](https://sublime.wbond.net/installation#Manual)

一些必备的package:
>
    Sublimerge           # 文件比对工具
    SideBarEnhancements  # 侧边栏增强
    Alignment            # 默认快捷键`ctrl+alt+a`
    Rubocop              # coding style检查工具, 需要先`gem install rubocop` 
  
若要进行BDD, 可以安装下列package:
>
    Behat  
    Sublime-Behat-Snippet  
    Cucumber  
    Gherkin[Cucumber] Formatter

关于sublime text2下的中文输入, 可以参照上一篇
  
### 服务器环境搭建
安装imageMagicksudo 
    $ sudo apt-get install imagemagick libmagickwand-dev
  
安装ffmpeg[参考这里](http://ffmpeg.org/trac/ffmpeg/wiki/UbuntuCompilationGuide)

安装nginx(带flv/mp4/rtmp/rtmp-push-stream模块)    
下载nginx源代码 1.3.10 至`/home/hanbing/nginx-1.3.10`  
下载nginx-rtmp模块源代码 至 `/home/hanbing/gao.nginx-rtmp-module`  
下载pcre 8.31至`/home/hanbing/pcre-8.31`
    $ rvmsudo passenger-install-nginx-module
按Enter确定---->选择2，自定义安装---->指定nginx源码位置：`/home/hanbing/nginx-1.3.10` ---->默认安装位置不变，按回车继续---->  复制粘贴下面的配置：
    --with-http_flv_module --with-http_mp4_module --add-module=/home/hanbing/nginx-push-stream-module --add-module=/home/hanbing/gao.nginx-rtmp-module --with-pcre=/home/hanbing/pcre-8.31
按回车键继续---->回车键确认配置正确，开始编译（若不正确输入no,回车，重新加配置）---->等编译---->回车over  
  
    
  
  