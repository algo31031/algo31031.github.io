---
layout: post
title: "多台电脑向同一Octopress Blog发表文章(Github Pages)"
date: 2014-4-21 11:07:21 +0800
comments: true
categories: Octopress
---
前段时间在利用Github Page搭了个Octopress, 顺便把以前网易记的一些东西也搬过来  
不过由于是用公司电脑搭的, 回到家里clone下源码, 改些东西再提交, 每次都会报master分支下文件冲突, 很是郁闷  
今天有幸google到了[Scott Cheng的这篇Blog](http://scottcheng.com/blog/2012/11/setting-up-existing-octopress-blog-on-a-new-computer/), 茅塞顿开, 感谢Scott  
以下是具体解决办法:

{% codeblock lang:bash %}
$ git clone git@github.com:algo31031/algo31031.github.io.git
$ cd algo31031.github.io
$ git checkout source
$ git branch -d master
$ bundle install 
$ bundle exec rake setup_github_pages
$ cd _deploy
$ rm -f index.html
$ git pull origin master
$ cd ..
$ git checkout source
{% endcodeblock %}

之后修改`./.git/config`, 在`[remote "origin"]`里加一行`user = algo31031`

搞定, 可以随意发表blog了