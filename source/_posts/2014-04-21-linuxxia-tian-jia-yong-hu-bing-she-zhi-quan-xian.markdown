---
layout: post
title: "linux下添加用户并设置权限"
date: 2014-03-16 14:37:26 +0800
comments: true
categories: linux
---

创建group(admin)
    $ sudo groupadd admin

向指定group(admin)添加user(vagrant)
    $ sudo useradd -g admin vagrant
设置user密码 
    $ sudo passwd vagrant 

更改user的group
    $ sudo usermod -G 组名 用户名

修改sudoers
    $ sudo visudo

会打开`/etc/sudoers`, 在里面添加:
    vagrant  ALL=(ALL:ALL)  NOPASSWD: ALL
    # 向vagrant用户添加权限, `NOPASSWD`确保vagrant执行任何命令时不需要输入密码
    %admin ALL=(ALL:ALL) ALL
    # 向admin组添加权限

四个ALL分别的含义:
>
  The 1st "ALL" indicates that this rule applies to all hosts.  
  The 2nd "ALL" indicates that the demo user can run commands as all users.  
  Th3 3rd "ALL" indicates that the demo user can run commands as all groups.  
  The 4th "ALL" indicates these rules apply to all commands.  

参考文章:
[http://linux.vbird.org/linux_basic/0410accountmanager.php#users_adduser](http://linux.vbird.org/linux_basic/0410accountmanager.php#users_adduser)
[https://www.digitalocean.com/community/articles/how-to-edit-the-sudoers-file-on-ubuntu-and-centos](https://www.digitalocean.com/community/articles/how-to-edit-the-sudoers-file-on-ubuntu-and-centos)