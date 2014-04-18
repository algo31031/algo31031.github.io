---
layout: post
title: "通过vagrant和knife-solo搭建AWS_EC2开发环境"
date: 2014-04-13 16:45:26 +0800
comments: true
categories: [部署, AWS]
---

###安装vagrant-aws插件

{% codeblock lang:bash %}
$ vagrant plugin install vagrant-aws
{% endcodeblock %}

###配置vagrant

{% codeblock lang:bash %}
$ mkdir aws
$ cd aws
$ vagrant init
$ vi Vagrantfile
{% endcodeblock %}

修改配置文件`Vagrantfile`, 添加下列内容:

{% codeblock lang:ruby %}
config.vm.define :osapp01d do |osapp01d|

  osapp01d.vm.provider :aws do |aws, override|

    aws.access_key_id = 您的aws access key id
    aws.secret_access_key = 您的aws secret access key

    aws.instance_type = 您的EC2 instance类型
    aws.ami = "ami-9ffa709e" #cent-os 6.4 64位
    aws.region = "ap-northeast-1" #东京主机
    aws.security_groups = 您的security group id
    aws.subnet_id = 您的subnet id
    aws.tags = {
      'Name' => "osapp01d"
    }
    aws.elastic_ip = true #为EC2 instance绑定静态IP

    override.ssh.username = "root"
    aws.keypair_name = 您的key pair name
    override.ssh.private_key_path = .pem私钥的位置

    override.vm.box = "dummy"
    override.vm.box_url = "https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box"  

    override.vm.synced_folder "./", "/vagrant", disabled: true 
  end
end
{% endcodeblock %}

参数的含义和赋值请[参考这里](https://github.com/mitchellh/vagrant-aws)   
请把相关参数替换成您的实际值

PS:关于各linux发行版的默认user

For Amazon Linux, the default user name is ec2-user. For RHEL5, the user name is often root but might be ec2-user. For Ubuntu, the user name is ubuntu. For SUSE Linux, the user name is root. Otherwise, check with your AMI provider.

###创建 EC2 instance 并启动

{% codeblock lang:bash %}
vagrant up --provider=aws osapp01d
{% endcodeblock %}

如果您使用的是gnome-terminal, 请注意要设置为[以登录shell方式运行命令(Run command as login shell)](http://worldofgnome.org/understanding-run-command-as-a-login-shell-option-in-gnome-terminal/)  
`Edit > Profile Preferences > Title and Command > Run command as a login shell`    

###登录EC2 instance, 添加vagrant用户

{% codeblock lang:bash %}
vagrant ssh osapp01d
{% endcodeblock %}

以下代码在EC2 inscatnce里执行 

{% codeblock lang:bash %}
$ groupadd admin
$ useradd -g admin vagrant
$ passwd vagrant
{% endcodeblock %}

设置vagrant用户的密码
{% codeblock lang:bash %}
$ visudo
{% endcodeblock %}

在最末添加一行:

`%admin ALL=(ALL:ALL) ALL`

如需使用同步文件夹，需将`Defaults    requiretty`这行注释掉  
保存退出

###向EC2 instance安装cookbooks
{% codeblock lang:bash %} 
$ mkdir Cookbooks 
$ cd Cookbooks
$ knife solo prepare vagrant@xx.xx.xx.xx(您的EC2 instance的public ip)
$ knife solo cook vagrant@xx.xx.xx.xx(您的EC2 instance的public ip) nodes/osapp01d,json
{% endcodeblock %}

###关于文件夹同步  
如果您希望使用同步文件夹,  
请将Vagrantfile里的`override.vm.synced_folder "./", "/vagrant", disabled: true`注释掉  
并且重启EC2 instance
{% codeblock lang:bash %}
vagrant reload osapp01d
{% endcodeblock %}

请注意这一步要在**EC2 instance创建**完成后,  
并且您已经将`Defaults    requiretty`注释掉后才可执行.   
否则会发生问题vagrant无法正常启动, [详情见这里](https://github.com/mitchellh/vagrant/issues/1659)       

