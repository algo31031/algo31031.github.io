---
layout: post
title: "设置swap分区"
date: 2014-03-21 14:46:08 +0800
comments: true
categories: linux
---
** 首先吐槽下, AWS EC2 t1.micro 不要随便开swap, 不然EBS会有海量IOs然后账单飞涨 **

创建swap文件件
    $ sudo /bin/dd if=/dev/zero of=/var/swap.1 bs=1M count=1024
`bs=1M`表示采用1M大小的block size  
`count=1024`表示需要1024个block  

将文件格式化为swap文件格式
    $ sudo mkswap /var/swap.1

挂载swap分区
    $ sudo swapon /var/swap.1

在/etc/fstab里添加`/var/swap.1 swap swap defaults 0 0`, 使swap开机之后能自动挂载:

可以用`$ sudo mount -a`测试一下fstab文件是否有问题, 一旦改错可能会导致无法启动

若无问题,重启

若要关闭swap
    $ sudo swapoff /var/swap.1

若以后不再使用这个swap， 可以直接把这个文件删掉， 然后把/etc/fstab里的内容注释掉

[参考](http://www.the-tech-tutorial.com/?p=1408)

