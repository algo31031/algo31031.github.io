---
layout: post
title: "为apache添加反向代理,在https server下使用faye"
date: 2014-05-05 15:34:58 +0800
comments: true
categories: [rails, deploy]
---
前段时间为正在着手的项目添加了基于faye的即时聊天功能    
开发时参照了[RailsCasts的教程](http://railscasts.com/episodes/260-messaging-with-faye?view=asciicast), 还算比较顺利  
但是部署到测试服务器时候, 发生了些问题

测试服务器使用了https server, 所有请求都走在https下  
但是我的faye server跑在http下, 导致faye.js无法加载, faye client也无法创建  

几经折腾, 最后采用apache添加了reverse proxy办法, 把https下的对faye的请求代理到http下faye server实际位置  
以下为具体操作

为apache添加反向代理
``` bash 加载proxy和proxy_http module
$ sudo a2enmod proxy
$ sudo a2enmod proxy_http
```

修改apache.conf
``` apacheconf apache.conf
<VirtualHost *:80>
    ServerName myapp.example.com
    RewriteEngine On  
    RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI} [L,R=permanent]
</VirtualHost>


# Provide an HTTPS entry point as well but one that will not spin up a second rails instance
# but rather redirect traffic accordingly
<VirtualHost *:443>
    ServerName myapp.example.com
    DocumentRoot /home/hanbing/work/svn/college/trunk/public
    <Directory /home/hanbing/work/svn/college/trunk/public>
      Require all granted 
      Order allow,deny
      AllowOverride all
      Allow from all
      # MultiViews must be turned off.
      #Options -MultiViews
    </Directory>
   
    SSLEngine on 
    SSLOptions +StrictRequire 
    SSLCipherSuite ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP:+eNULL
    SSLCertificateFile /etc/apache2/certs/myapp.example.com.crt
    SSLCertificateKeyFile /etc/apache2/certs/myapp.example.com.key     

    ProxyPass /faye.js http://127.0.0.1:9292/faye.js
    ProxyPassReverse /faye.js http://127.0.0.1:9292/faye.js

    ProxyPass /faye http://127.0.0.1:9292/faye
    ProxyPassReverse /faye http://127.0.0.1:9292/faye
</VirtualHost>
```

在controller里添加
``` ruby
@faye_server =  if request.ssl?
                  request.protocol << request.host_with_port
                else
                  request.protocol << request.host << ":9292"
                end
```

创建faye client
``` erb
<%= javascript_include_tag @faye_server << "/faye.js" %>
<script language="javascript" type="text/javascript">
  var faye = new Faye.Client("<%= @faye_server %>/faye");

  faye.subscribe('/chat/new', function (data) {
    $("chat_list").insert({bottom: data["chat_log"].toString()});

    var post = $("chat_list").childElements().last();

    if(data["user_id"].toString() != "<%= current_user.id %>"){
      post.removeClassName("my-post");
      post.addClassName("member-post")
    }
    var objDiv = $("chat_list");
    objDiv.scrollTop = objDiv.scrollHeight;    
    new Effect.Highlight(post.id);
    $("errors").hide();

  });

</script>
```