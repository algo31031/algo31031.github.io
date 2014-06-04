---
layout: post
title: "使用faye的Extensions和Monitoring实时记录用户在线状况"
date: 2014-06-04 15:14:02 +0800
comments: true
categories: rails
---

faye的client如果想要接收到某个channel的聊天信息, 需要先subscribe这个channel
``` js
var faye = new Faye.Client("localhost:9292/faye");

faye.subscribe('/chat/channelXX', function (data) {
  XXXXXXXXXXXXXx
})
``` 
在进行subscribe时候, 会使用特定的channel `/meta/subscribe`,  
并且faye server会对faye client分配一个唯一的client_id

类似的, 当faye client执行unsubscribe和disconnect时,  
也会使用`/meta/unsubscribe`和`/meta/disconnect`


于是, 可以在faye的browser client端添加扩展  
使其在使用channel `/meta/subscribe`时, 传入ActiveRecord的相关id
``` erb
faye.addExtension({
  outgoing: function(message, callback) {
    if (message.channel == '/meta/subscribe') {
      message.data = {user_id: <%= current_user.id %>, chat_team_id: <%= @chat_team.id %>};
    }
    callback(message);
  }
}); 
```

之后, 与faye server端添加扩展, 通过传入的数据, 找到相应ActiveRecord记录, 并更改其状态为在线  
同时还要记录下其client_id, 作为此user离线时再次找到此人的依据
``` ruby
class  MarkOnline
  def incoming(message, callback)

    if message['channel'] == '/meta/subscribe'
      UserChatTeam.mark_online( message['data']['user_id'], 
                                message['data']['chat_team_id'],
                                message['clientId']) 
    end

    callback.call(message)
  end

end

faye_server = Faye::RackAdapter.new(:mount => '/faye', :timeout => 45)
faye_server.add_extension(MarkOnline.new)
```

为了能在rack程序里使用ActiveRecord, 还需要在faye.ru文件里添加以下内容
``` ruby
require 'active_record'
require 'mysql'
require 'yaml'
require File.expand_path('../app/models/user_chat_team.rb', __FILE__)

environment = ENV['RACK_ENV'] || 'production'
dbconfig    = YAML.load(File.read('config/database.yml'))
ActiveRecord::Base.establish_connection(dbconfig[environment])
```

接下来在faye server里添加monitor, 监视unsubscribe和disconnect事件  
并根据之前记录的client_id将相关记录标记为离线, 同时清除client_id
``` ruby
faye_server.on('unsubscribe') do |client_id, channel|
  UserChatTeam.mark_offline(client_id)  
end

faye_server.on('disconnect') do |client_id|
  UserChatTeam.mark_offline(client_id)   
end
```

** 可能会出现的问题 **  
同一user对相同channel如果打开多个窗口  
当其关闭其中一个窗口后, 即会判断为此user已离线

** 参考 **  
[faye monitoring](http://faye.jcoglan.com/ruby/monitoring.html)  
[https://github.com/eoecn/faye-online](https://github.com/eoecn/faye-online)  
[Faye Exensions: Tracking Users in a Chat Room](http://blog.edweng.com/2012/06/02/faye-extensions-tracking-users-in-a-chat-room/)

``` ruby faye.ru
require 'faye'
require File.expand_path('../config/initializers/faye_token.rb', __FILE__)
require 'active_record'
require 'mysql'
require 'yaml'

# using 'acts_as_paranoid' as a plugin in 'vendor/plugins'
RAILS_ENV = ENV['RACK_ENV'] 
load 'active_record/associations.rb' 
require File.expand_path('../vendor/plugins/acts_as_paranoid/lib/caboose/acts/paranoid.rb', __FILE__)
require File.expand_path('../vendor/plugins/acts_as_paranoid/lib/caboose/acts/belongs_to_with_deleted_association.rb', __FILE__)
require File.expand_path('../vendor/plugins/acts_as_paranoid/lib/caboose/acts/has_many_through_without_deleted_association.rb', __FILE__)
require File.expand_path('../vendor/plugins/acts_as_paranoid/init.rb', __FILE__)
require File.expand_path('../app/models/user_chat_team.rb', __FILE__)

environment = ENV['RACK_ENV'] || 'production'
dbconfig    = YAML.load(File.read('config/database.yml'))
ActiveRecord::Base.establish_connection(dbconfig[environment])

class ServerAuth
  def incoming(message, callback)

    if message['channel'] !~ %r{^/meta/}
      if message['ext']['auth_token'] != FAYE_TOKEN
        message['error'] = 'Invalid authentication token.'
      else
        message['ext'].delete('auth_token')
      end
    end

    callback.call(message)
  end
end

class  MarkOnline
  def incoming(message, callback)

    if message['channel'] == '/meta/subscribe'
      UserChatTeam.mark_online( message['data']['user_id'], 
                                message['data']['chat_team_id'],
                                message['clientId']) 
    end

    callback.call(message)
  end

end


faye_server = Faye::RackAdapter.new(:mount => '/faye', :timeout => 45)
faye_server.add_extension(ServerAuth.new)
faye_server.add_extension(MarkOnline.new)

faye_server.on('unsubscribe') do |client_id, channel|
  UserChatTeam.mark_offline(client_id)  
end

faye_server.on('disconnect') do |client_id|
  UserChatTeam.mark_offline(client_id)   
end

run faye_server
```

``` erb faye browser client
var faye = new Faye.Client("<%= @faye_server %>/faye");
faye.disable('websocket');

faye.addExtension({
  outgoing: function(message, callback) {
    if (message.channel == '/meta/subscribe') {
      message.data = {user_id: <%= current_user.id %>, chat_team_id: <%= @chat_team.id %>};
    }
    callback(message);
  }
}); 

faye.subscribe('/chat/<%= @chat_team.channel %>', function (data) {
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
```