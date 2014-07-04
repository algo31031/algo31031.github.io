---
layout: post
title: "几个鲜为人知的ActiveRecord Associations小技巧 (译)"
date: 2014-07-04 10:08:40 +0800
comments: true
categories: [rails, quote]
---
原文出自[gotealeaf.com](http://www.gotealeaf.com/blog/ruby-on-rails-activerecord-association-features/), 感谢作者Steve Turczyn

作为一个Rails开发者, 与ActiveRecord associations打交道实属家常便饭. 但其中若干特性, 却非尽人皆知.

### 自定义查询

假如你在开发一个允许发表评论的博客, 保不齐会遇到各种不和谐的言论充斥于评论间(这里毕竟是互联网). 为了保证只显示经核准的评论, 你要在comments表里加一个`boolean`型字段:`approved`.

对于每篇博客, 由于只想取出审核过的评论(即那些被管理员标记为通过的), 所以需要自定义查询. 

```ruby
# app/models/post.rb
class Post < ActiveRecord::Base
  has_many :approved_comments, -> { where approved: true }, class_name: 'Comment'
  ...
```

现在, 当你需要取出某篇博客里审核过的评论时, 只需要使用`my_post.approved_comments`.  

### 扩展

有可能你还是需要有个选项能调出一篇博客的全部评论, 或者只看在今天发的那些. 这时你可以利用扩展功能, 在某个特定的关系上追加一个自定义方法.

```ruby
# app/models/post.rb
class Post < ActiveRecord::Base
  has_many :comments do
    def today
      where("created_at >= ?", Time.zone.now.beginning_of_day)
    end
  end
  ...
```

这样一来, 除了原本的`my_post.comments`, 你还能通过诸如`my_post.comments.today`这种调用让检索更确切.  

### 回调

在记录被添加或移除之前与之后, 自动的调用方法, 这个可以有.

假如你有一个建筑项目(BuildingProject), 里面有许多工人(Worker). 每次增加一个工人, 都需要重新计算项目预算和完工日期的变化, 而这些有又可能与工人类型工作经验等等息息相关...好个麻烦的方法, 要能在添加完工人后自动调用就好了.

```ruby
# app/models/building_project.rb
class BuildingProject < ActiveRecord::Base
  has_many :workers, after_add: :recalculate_project_status
  ...
  def recalculate_project_status(newly_added_worker)
    ...
  end
  ...
```

### inverse_of

`inverse_of`可以很方便的使相互关联的对象从任意方向被访问到. `my_post.comments`和`my_comment.post`. 当然, 在`Post`里指定`has_many :comments`并且在`Comment`里指定`belongs_to :post`之后, 便已经可以得到上述2个关系. 但是加上`inverse_of`之后, 会让rails确保关系里的对象乃是相同的对象. 

`inverse_of`指定方法如下:

```ruby
# app/models/post.rb
class Post < ActiveRecord.Base
 has_many :comments, inverse_of: post
 ...
```

现在先去掉`inverse_of`, 如果你这样这样这样...

```ruby
post = Post.first
post.update_attribute(:importance, false)
comment = post.comments.first
working_post = comment.post
working_post.update_attribute(:importance, true)
post.importance?
=> false
```

喔! 系统咋还是觉得你的blog不给力...这是因为`comment.post`执行了一次单独的DB查询, 得到的对象虽然对应DB里同一条Post记录,但却是不同的实例. 而原本那条post记录的对象还是不知道自己在DB的状态已经被改变了.

那么让我们把`inverse_of`加回去, 重新执行刚才的代码:

```ruby
post = Post.first
post.update_attribute(:importance, false)
comment = post.comments.first
working_post = comment.post
working_post.update_attribute(:importance, true)
post.importance?
=> true
```

好多了! 加上`inverse_of`之后, `comment.post`不再做DB查询, 而是直接使用已被加到内存里的post对象的实例. 换言之, 这时的`working_post`就是上面的`post`, 所以对`working_post`的改变会直接反应在`post`上.