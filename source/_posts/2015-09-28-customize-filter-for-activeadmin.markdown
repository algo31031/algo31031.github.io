---
layout: post
title: "在ActiveAdmin下自定义filter"
date: 2015-09-28 09:20:19 +0800
comments: true
categories: rails
---
rails4之后，之前的`model`里加入`search_methods :filter_name`的做法已经失效，需修改为：

``` ruby
  class PromoCode < ActiveRecord::Base
    scope :by_tag_ids, ->(ids) { with_tags(ids) }

    def self.ransackable_scopes(auth_object = nil)
      [:by_tag_ids]
    end
  end
```

``` ruby
  ActiveAdmin.register PromoCode do
    filter :by_tag_ids, as: :string, label: '标签ID'
  end
```
