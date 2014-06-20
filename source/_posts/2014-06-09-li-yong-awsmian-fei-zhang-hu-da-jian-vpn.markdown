---
layout: post
title: "利用AWS免费账户搭建PPTP VPN"
date: 2014-06-09 14:35:19 +0800
comments: true
categories: AWS
---

** 首先, 你要有一个信用卡...  **
或者似乎有人说可以在淘宝购买虚拟信用卡也是可行的, 但是里面余额要大于1美金

** 另外有人反应说注册时候被扣掉1美金 **,  
是因为首次注册时, ASW会为了验证信用卡有效性冻结1美金  
被冻结的钱在服务到期后会返还  
另外好像双币信用卡不会有这个问题


0. ### 注册 AWS 账户
    进入[AWS主页](http://aws.amazon.com/), 按照指示完成注册.
    
    需要注意注册过程会有一步 Identity Verification by Telephone,  
    需要输入电话号码然后点击"Call Me Now", 稍等片刻 Amazon 会有电话拨过来, 

    ![call_me_now](http://blog-banban.qiniudn.com/call_me_now.png)

    接到电话后在电话上输入您屏幕上的 "Your Pin:" 后标示的四位数字即可.

    之后会要求选择AWS Support Plan, 当然选Basic (Free)

    注册成功之后会进入Thank you页面, 选择启动AWS管理控制台.  

    ![launch_console](http://blog-banban.qiniudn.com/launch_console.png)

    之后选择"登录AWS控制台"按钮(右上方, 比较醒目的橙色)
    
    另外注册后您可能会受到从aws-verification@amazon.com发来的email,  
    提醒你没通过验证无法使用Amazon Route 53 和 CloudFront(分别是AWS的NS和CDN服务).  

    如果不打算用(反正也都收钱的...), 直接忽略即可

    如果想试一下这2个服务  
    您可以提供 valid business URL 并使用此 URL 向aws-verification@amazon.com发一封确认邮件.    
    邮件可用以简单些, 内容如下即可(请注意使用您自己公司的site-url和email-address):
    
    >
      site url:         http://www.e-trust.com.cn/    
      email address:    han.bing@.e-trust.com.cn  
      account:          algo31031@gmail.com  
   
    同时您还需要更改您的个人信息, 在 Web Site URL: 填入您公司的首页,  
    Web Site URL: 的具体位置见下图:
    
    ![web_site_url](http://blog-banban.qiniudn.com/web_site_url.png)
    
    邮件发送之后稍等片刻, 会收到Amazon的确认信息.
    
1. ### 启动并设置AWS EC2
    0. ** 选择region **
        部署区域, 简易选择东京

        ![select_region](http://blog-banban.qiniudn.com/select_region.png)

    1. ** 创建下载 Key Pairs **

        进入控制台后, 默认就会来到EC2 Dashbord页, 在这里选择Key Pairs
        创建 Key Pairs
        
        ![key_pairs](http://blog-banban.qiniudn.com/key_pairs.png)

        完成后会得到一个.pem文件, 把他移动到`~/.ssh`目录下  
        并更改权限为600

            mv ~/文件所在位置/algo31031.pem ~/.ssh/
            chmod 600 ~/.ssh/algo31031.pem
        
        请妥善保管您的.pem文件, 若遗失您可以删除之前的key_pairs并创建新的  
        如果您希望使用自己的ssh key, 也可以选择"Import Key Pair"                         

    2. ** 启动EC2实例 **

        回到EC2 Dashbord页, 在这里选择Launch instance

        ![launch_ec2_instance](http://blog-banban.qiniudn.com/launch_ec2_instance.png)

        选择需要的系统, 这里我们选用64位的ubuntu 14.04 PV了  
        需要注意下如果没有free tier eligible标志的是需要收费的

        ![select_ami](http://blog-banban.qiniudn.com/select_ami.png)

        选择实例类型时候, 要选择Micro instances  
        记住这一点: 没有Free tier eligible都是要收钱的(有这个的用超了也要收钱...)
        之后一路Next(Add Storage那里可以适当加点, 但是不要超过30G), 一直到Configure Security Group
        这里除了默认的ssh 22 端口外, 还要开放1723端口, 否则连不上VPN

        ![add_security_role](http://blog-banban.qiniudn.com/add_security_role.png)

        在launch EC2实例之前, 会有一步让你选择key pair, 用刚才创建的那个即可

        ![select_key_pair](http://blog-banban.qiniudn.com/select_key_pair.png)

        至此, 已经有了一台AWS EC2可以用来折腾了

        ![launch_status.png](http://blog-banban.qiniudn.com/launch_status.png)

2. ### 登录EC2配置安装PPTV VPN
    
    0. ** 为EC2实例绑定Elastic IP **
        创建完的EC2实例虽然有自己的公网IP, 但是每次重新启动都会变化  
        所以需要为其绑定Elastic IP

        ![add_elastic_ip](http://blog-banban.qiniudn.com/add_elastic_ip.png)

        申请完之后, 将其与刚才创建的EC2实例关联

        ![associate_ec2_instance](http://blog-banban.qiniudn.com/associate_ec2_instance.png)

    1. ** 通过ssh登录EC2实例 **

        修改`~/.ssh/config`, 在里面添加如下内容, 注意替换HostName成自己的Elastic IP  
        并把IdentityFile改成自己机器上.pem文件的路径

            Host aws_ec2
              HostName 54.199.xxx.xxx
              User ubuntu
              IdentityFile /home/hanbing/.ssh/algo31031.pem

        打开终端, 使用`ssh aws_ec2`登录EC2实例

    2. ** 在EC2实例下安装PPTP VPN **

        具体内容可以[参考这篇文章](http://www.yzhang.net/blog/2013-03-07-pptp-vpn-ec2.html), 以下内容只是对原文重点部分摘录并稍作修改以适应ubuntu

        安装pptpd

        `sudo apt-get install pptpd`

        修改`/etc/pptpd.conf`文件, 在最先面添加以下2行

            localip     192.168.9.1
            remoteip    192.168.9.11-30

        之后修改`/etc/ppp/options.pptpd`文件, 加上谷歌的dns

            ms-dns    8.8.8.8
            ms-dns    8.8.4.4

        接下来修改`/etc/ppp/chap-secrets`文件, 配置你自己VPN的用户名/密码, 格式如下:

            <username> pptpd <passwd> *

        修改`/etc/sysctl.conf`文件, 添加以下内容(默认里面有这行, 把注释去掉也可)

            net.ipv4.ip_forward=1

        执行`sudo /sbin/sysctl -p`重新加载配置

        启用`iptables`的NAT configuration

            sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

        为了保证每次EC2实例重启后NAT configuration能启动, 还要修改`/etc/rc.local`文件, 
        在`exit 0`这行上面加上

            iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

        最后重启pptpd服务

            sudo /etc/init.d/pptpd restart

3. ### 连接VPN

    ** 注意 ** AWS免费账户每月只有** 15G **免费流量,  用超了要** 从信用卡扣钱的 **

    0. ** ubuntu用户 **

        ![vpn1.png](http://blog-banban.qiniudn.com/vpn1.png)

        ![vpn2.png](http://blog-banban.qiniudn.com/vpn2.png)

        ![vpn3.png](http://blog-banban.qiniudn.com/vpn3.png)

    1. ** Mac用户 ** 
    
        我没有mac, 图直接从别人那里偷的...

        ![mac_vpn1.png](http://blog-banban.qiniudn.com/pptp-vpn-mac-1.png)
        
        ![mac_vpn2.png](http://blog-banban.qiniudn.com/pptp-vpn-mac-2.png)

4. ### 更换ip

    需要先将原有的elastic ip从ec2实例解除绑定, 然后释放这个ip  
    ** 注意 ** 一定要把不用的ip释放掉, 不然超过15分钟后就要扣钱的(只有绑定到运行中的ec2上才不扣钱)

    ![reset_ip](http://blog-banban.qiniudn.com/reset_ip_1.png)

    之后按照之前的流程重申请一个Elastic IP并绑定到EC2实例即可

    更换完IP之后记得要把svn连接信息里的地址一并替换掉