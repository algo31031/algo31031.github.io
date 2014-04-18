---
layout: post
title: "使用vagrant和knife-solo配合Vitrualbox搭建测试环境"
date: 2014-04-12 19:21:22 +0800
comments: true
categories: [部署, ubuntu]
---
###下载安装 Virtualbox
这里可以直接用ubuntu软件中心提供的版本    
其他系统可以再下面找  
[下载Virtualbox](https://www.virtualbox.org/wiki/Downloads)

###下载安装 vagrant
[点击进入下下载列表](https://www.vagrantup.com/downloads.html)  
注意低版本的vagrant提供gem安装方式, 目前已不支持, 请不要采取此种方式, 详见[此处](http://mitchellh.com/abandoning-rubygems)    

###下载安装 vagrant 的 centos box
[点击下载box](http://developer.nrel.gov/downloads/vagrant-boxes/CentOS-6.4-x86_64-v20130731.box)  
安装box    
    $ vagrant box add centos-6.4 /home/hanbing/Downloads/CentOS-6.4-x86_64-v20130731.box  
可以在下面找到其他操作系统的box  
[vagrantbox](http://www.vagrantbox.es/)  

###配置vagrant

{% codeblock lang:bash %}
$ cd ~/
$ mkdir vagrant
$ cd vagrant
$ vagrant init
$ vi Vagrantfile
{% endcodeblock %}
    
把下面内容添加到 `Vagrantfile`  

{% codeblock lang:ruby %}
config.vm.box = "centos-6.4"
config.vm.network :private_network, ip: "192.168.33.10"
config.vm.host_name = "oshost"
{% endcodeblock %}
  
设置共享文件夹(若不配置,默认把host的Vagrantfile所在目录共享到guest的/vagrant),  
第一个参数是host machine的待共享文件夹, 可以用相对路径.   
第二个参数是gues tmachine的共享文件夹, 必须为绝对路径, 若不存在会自动创建一个.    
其他支持的options可以参看[这里](http://docs.vagrantup.com/v2/synced-folders/basic_usage.html)  
`config.vm.synced_folder "/home/hanbing", "/vagrant"`

有时默认的共享文件夹会有性能问题, 需要加参数nfs: true  
若要使用NFS文件夹, host machine需要安装了`nfsd`  
另外注意在Windows下不支持NFS, 所以Windows会自动忽略nfs参数  
`config.vm.synced_folder ".", "/vagrant", id: "vagrant-root", nfs: true`

配置好后可以通过`vagrant up`启动vm  
**Vagrant常用命令**
{% codeblock lang:bash %}
$ vagrant up 开机 
$ vagrant ssh 登⼊  
$ vagrant suspend 暂停  
$ vagrant halt 关机
$ vagrant destroy 刪除 
{% endcodeblock %}    

###插曲 启动时报错

{% codeblock lang:bash 报错信息 %}
hanbing@hanbing-System-Product-Name:~/vagrant$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Clearing any previously set forwarded ports...
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
    default: Adapter 2: hostonly
==> default: Forwarding ports...
    default: 22 => 2222 (adapter 1)
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: Error: Connection timout. Retrying...
    default: Error: Connection timout. Retrying...
    default: Error: Connection timout. Retrying...
    default: Error: Connection timout. Retrying...
    default: Error: Connection timout. Retrying...
    default: Error: Connection timout. Retrying...
    default: Error: Connection timout. Retrying...
    default: Error: Connection timout. Retrying...
    default: Error: Connection timout. Retrying...
    default: Error: Connection timout. Retrying...
    default: Error: Connection timout. Retrying...
    default: Error: Connection timout. Retrying...
    default: Error: Connection timout. Retrying...
    default: Error: Connection timout. Retrying...
    default: Error: Connection timout. Retrying...
{% endcodeblock %}        

原本以为ssh的问题, 后来发现是centOS无法启动,直接用VirtualBox启动时有如下提示:
    VT-x/AMD-V 硬件加速已被启用, 但当前处于无效状态. 您虚拟电脑内的操作系统将无法检测到64位的处理器，因此也将无法启动.

    请确认在您电脑的BIOS中已启用 VT-x/AMD-V 支持.

解决办法:  
(1)进入BIOS设置,把Virtualization设为为Enabled(一般会在高级设置里)  
(2)虚拟机设置 > 系统 > 硬件加速 > 启用VT-x/AMD-V  
(3)重新执行 `$ vargant up`  
正常了^_^

{% codeblock lang:bash %}
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
    default: Adapter 2: hostonly
==> default: Forwarding ports...
    default: 22 => 2222 (adapter 1)
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: Error: Connection timout. Retrying...
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
==> default: Configuring and enabling network interfaces...
==> default: Mounting shared folders...
    default: /vagrant => /home/hanbing/vagrant
{% endcodeblock %}

**PS:**2013版Mac air下不存在这个问题

### 开始安装各个cookbook
执行`$ vagrant ssh-config --host oshost`  
会出现类似下面内容:

{% codeblock lang:bash %}
Host oshost
  HostName 127.0.0.1
  User vagrant
  Port 
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /home/hanbing/.vagrant.d/insecure_private_key
  IdentitiesOnly yes
  LogLevel FATAL
{% endcodeblock %}  

将其加入`~/.ssh/config`内, 以后使用`$ ssh oshost`可以直接连入guest machine  
    $ vagrant ssh-config --host oshost >> ~/.ssh/config

除此以外，使用`vagrant ssh [主机名]`也可连入guest machine  
    $ vagrant ssh oshost

**创建kitchen, 下载cookbooks**
    $ mkdir Cookbooks
    $ cd Cookbooks`  

安装gem
    $ gem install librarian-chef
    $ gem install knife-solo  

在当前目录生成一个kitchen　　
    $ knife solo init .

在kitchen里生成Cheffile
    $ librarian-chef init  

编辑Cheffile添加需要的cookbooks  
下载Cheffile里要求的cookbooks到./cookbooks目录
    $ librarian-chef install

之后执行以下操作开始装机过程
{% codeblock lang:bash %}
$ knife solo prepare oshost
$ knife solo cook oshost
{% endcodeblock %}   

**knifo-solo命令**
{% codeblock lang:bash %}
$ knife solo init 目录                               新建一个符合chef结构的目录(kitchen)用以创建和储存recipes  
$ knife solo prepare user@host                      在给定的host安装chef
$ knife solo cook user@host [nodes/<hostname>.json] 将kitchen上传到指定的host并执行chef-solo
$ knife solo bootstrap user@host                    perpare+cook
{% endcodeblock %} 

###配置多台 vm

修改`Vagrantfile`:

{% codeblock lang:ruby %}
  config.vm.define :oshost do |oshost|
    oshost.vm.box = "centos-6.4"
    oshost.vm.network :private_network, ip: "192.168.33.10"
    oshost.vm.host_name = "oshost"
    oshost.vm.provider "virtualbox" do |v|
      v.name = "oshost"
    end        
  end
 
  config.vm.define :osdep01d do |osdep01d|
    osdep01d.vm.box = "centos-6.4"
    osdep01d.vm.network :private_network, ip: "192.168.33.11"
    osdep01d.vm.host_name = "osdep01d"
    osdep01d.vm.provider "virtualbox" do |v|
      v.name = "osdep01d"
    end        
  end  
{% endcodeblock %}


启动vm  
    $ vagrant up osdep01d

添加ssh host  
    $ vagrant ssh-config --host osdep01d >> ~/.ssh/config

向`osdep01d`安装cookbooks  
    $ knife solo bootstrap osdep01d

###参考文章
[HOWTO get started with chef, librarian-chef and vagrant](https://fak3r.com/2013/08/30/howto-get-started-with-chef-librarian-chef-and-vagrant/)  
[[教學]使用Vagrant練習環境佈署](http://gogojimmy.net/2013/05/26/vagrant-tutorial/)  
[[Rails佈署實戰教學]使用Chef-Solo一鍵安裝機器](http://gogojimmy.net/2013/06/01/Chef-Solo-Basic-with-Vagrant/)  
[Vagrant Provisioning, Chef solo and Librarian-chef](http://tumblr.nrako.com/post/22320729770/vagrant-chef-librarian)  

###相关资源
[VirtualBox](https://www.virtualbox.org/wiki/Downloads)  
[vagrantbox](http://www.vagrantbox.es/)  
[cookbooks](http://community.opscode.com/cookbooks)  