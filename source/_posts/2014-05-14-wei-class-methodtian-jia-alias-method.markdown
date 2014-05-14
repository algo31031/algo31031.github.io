---
layout: post
title: "为class method添加alias_method"
date: 2014-05-14 10:21:15 +0800
comments: true
categories: ruby
---
``` ruby 
class ChatTeamTopic < ActiveRecord::Base
  acts_as_paranoid

  ...

  class << self
    alias_method :orig_delete_all, :delete_all
     
    def delete_all(conditions = nil)
      clear_notifications(conditions) if conditions.present?
      orig_delete_all(conditions)
    end

    def clear_notifications(conditions)
      ...
    end
  end

  ...

end
```