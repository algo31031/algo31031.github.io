---
layout: post
title: "RVM is not a function"
date: 2012-03-26 10:47:59 +0800
comments: true
categories: rails
---
在主文件夹`.bashrc`中加入一行

    [[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm"

如果终端设置成[以登录shell方式运行命令(Run command as login shell)](http://worldofgnome.org/understanding-run-command-as-a-login-shell-option-in-gnome-terminal/)，则修改`.bash_profile`

重启终端