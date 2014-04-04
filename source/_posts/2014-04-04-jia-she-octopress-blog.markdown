---
layout: post
title: "架设Octopress Blog"
date: 2014-04-04 15:48:39 +0800
comments: true
categories: [部署 Octopress]
---
一直想弄个Blog整理些资料, 单靠胡乱的记到文件里放在Ubuntu One上， 既混乱查找起来也不方便。

碰巧前段时间因为工作需要开始研究AWS，遂去Godaddy注册了个域名，反正也不贵， AWS可以“免费”体验一年， 而Godaddy的.com域名一年只要56人民币，支持支付宝付款。 反正只是个blog不用考虑备案的诸多麻烦。

作为一个Rubist, 自然首选[Octrpress](http://octopress.org/)。

简单记下架设的过程，也顺便熟悉下Octopress用法。

##安装Octopress

注意需要Ruby 1.9.3及以上版本    

    $ git clone git://github.com/imathis/octopress.git octopress
    $ cd octopress
    $ gem install bundler
    $ bundle install

安装默认主题

    $ rake install

也可以在[这里](https://github.com/imathis/octopress/wiki/3rd-Party-Octopress-Themes)找到其他主题

选了一番决定用[Greyjoy](https://github.com/rezajatnika/greyjoy), 从github页看作者想必也是《冰与火之歌》的粉丝（Named after the messed up Theon Greyjoy.）。

安装greyjoy主题

    $ cd .themes
    $ git clone git://github.com/rezajatnika/greyjoy.git
    $ rake install[greyjoy]
    $ rake generate   

装完之后， 你就可以用`rake preview`预览下了

    $ rake preview

在浏览器地址栏输入`localhost:4000`就能看到        
  
##向Github User/Organization pages部署Octopress

在github上添加一个repo, 并用 `username.github.io` 这种格式命名。其中username是你在github上的用户名或组织名。

Github User/Organization pages会把你rope的master分支用作在`http://username.github.io`发布的内容，相应的我们可以把source 分支作为我们源码的分支。Octopress已经提供了一个内建的rake task来帮我们搞定上述操作。

    $ rake setup_github_pages

这个rake  task会要你输入一个Github repo的url， 我我们创建完的repo的https/ssh url任选其一粘过来回车即可（比如我的： `git@github.com:algo31031/algo31031.github.io.git`）

之后想要发表blog只要执行

    $ rake generate
    $ rake deploy

就可以把我们的blog提交到github repo的master分支下，过段时间就可以在`http://username.github.io`看到blog已经部署成功。

不要忘记把源码也一起放到github

    $ git add .
    $ git commit -m 'your message'
    $ git push origin source

##使用自己的域名

在source目录下创建一个名字是`CNAME`的文件，然后向CNAME里添加记录(二者择一即可, 或者干脆自己打开CNAME文件把`your-domain.com`或者`www.your-domain.com`考进去)

    $ echo 'your-domain.com' >> source/CNAME
    # OR
    $ echo 'www.your-domain.com' >> source/CNAME

然后去DSN服务商那里创建一条记录。如果用的子域名，只需要创建一个CNAME记录, 将子域名指向`http://username.github.io`。如果是主域名则需要A记录然后指定ip。