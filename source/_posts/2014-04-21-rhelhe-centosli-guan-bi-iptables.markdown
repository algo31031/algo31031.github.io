---
layout: post
title: "RHEL和CentOS里关闭iptables"
date: 2014-03-17 15:04:18 +0800
comments: true
categories: linux
---

查看iptables状态：
    $ sudo /etc/rc.d/init.d/iptables status

iptables开机自动启动： 
    $ sudo chkconfig iptables on
    $ sudo chkconfig iptables off

iptables关闭服务：
    $ sudo /etc/rc.d/init.d/iptables start
    $ sudo /etc/rc.d/init.d/iptables stop
