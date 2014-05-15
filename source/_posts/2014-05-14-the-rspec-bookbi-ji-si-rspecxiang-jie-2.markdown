---
layout: post
title: "The RSpec Book笔记(四) Code Examples(2)"
date: 2014-05-14 10:55:35 +0800
comments: true
categories: [test, RSpec]
---
### 12.4 Helper Methods

我们可以在example group中定义helper方法，这样定义的helper方法可以在example group中的每个code example里使用。

例如:

``` ruby
describe Thing do
  it "should do something when ok" do
    thing = Thing.new
    thing.set_status('ok')
    thing.do_fancy_stuff(1, true, :move => 'left', :obstacles => nil)
    ...
  end

  it "should do something else when not so good" do
    thing = Thing.new
    thing.set_status('not so good')
    thing.do_fancy_stuff(1, true, :move => 'left', :obstacles => nil)
    ...
  end
end
```    

可以把`thing = Thing.new`和`thing.set_status('ok')`部分提取出来,改成:

``` ruby
describe Thing do
  def create_thing(options)
    thing = Thing.new
    thing.set_status(options[:status])
    thing
  end

  it "should do something when ok" do
    thing = create_thing(:status => 'ok')
    thing.do_fancy_stuff(1, true, :move => 'left', :obstacles => nil)
    ...
  end

  it "should do something else when not so good" do
    thing = create_thing(:status => 'not so good')
    thing.do_fancy_stuff(1, true, :move => 'left', :obstacles => nil)
    ...
  end
end
```

还可以进一步提取为(但是个人感觉这么做意义不大, 纯属是为了使其更符合英语语法结构):

``` ruby
describe Thing do
  def given_thing_with(options)
    yield Thing.new do |thing|
      thing.set_status(options[:status])
    end
  end

  it "should do something when ok" do
    given_thing_with(:status => 'ok') do |thing|
      thing.do_fancy_stuff(1, true, :move => 'left', :obstacles => nil)
      ...
    end
  end

  it "should do something else when not so good" do
    given_thing_with(:status => 'not so good') do |thing|
      thing.do_fancy_stuff(1, true, :move => 'left', :obstacles => nil)
      ...
  end
end
```    

** 共享Helper Methods **

为了能够在example groups之间共享Helper Methods, 我们可以将Helper Methods定义在module里  
并在需要使用这些方法的时候, 在example groups里面include相应的module.  

例如:

``` ruby
module UserExampleHelpers
  def create_valid_user
    User.new(:email => 'email@example.com', :password => 'shhhhh')
  end

  def create_invalid_user
    User.new(:password => 'shhhhh')
  end
end

describe User do
  include UserExampleHelpers

  it "does something when it is valid" do
    user = create_valid_user
    # do stuff
  end

  it "does something when it is not valid" do
    user = create_invalid_user
    # do stuff
  end
```

如果某个module中的helper methods, 在所有的example group里都用得上  
还可以把这个module通过配置全局加载

``` ruby
RSpec.configure do |config|
  config.include(UserExampleHelpers)
end
```

### 12.5 Shared Examples

有时我们想要某个class的多个不同实例表现出相同的行为  
这是我么可以采用shared example group将其行为描述出来  
然后在需要的地方include这个example group

这里我们会用到2个方法:

** shared_examples_for **  用来用来声明shared example group
``` ruby
shared_examples_for "any pizza" do
  it "tastes really good" do
    @pizza.should taste_really_good
  end
end
```


** it_behaves_like ** 用来include声明过的shared example group
``` ruby
describe "New York style thin crust pizza" do
  before(:each) do
    @pizza = Pizza.new(:region => 'New York', :style => 'thin crust')
  end

  it_behaves_like "any pizza"

  it "has a really great sauce" do
    @pizza.should have_a_really_great_sauce
  end
end

describe "Chicago style stuffed pizza" do
  before(:each) do
    @pizza = Pizza.new(:region => 'Chicago', :style => 'stuffed')
  end

  it_behaves_like "any pizza"

  it "has a ton of cheese" do
    @pizza.should have_a_ton_of_cheese
  end
end
```

`it_hehaves_like`方法会生成一个名为behaves like xxx的嵌套group(nested example group)  
传入`it_hehaves_like`的代码块会在这个嵌套group里执行  

于是, 上述代码运行时会得到以下输出内容:

```
New York style thin crust pizza
  has a really great sauce
  behaves like any pizza
    tastes really good
    is available by the slice

Chicago style stuffed pizza
  has a ton of cheese
  behaves like any pizza
    tastes really good
    is available by the slice
```

### 12.6 Nested Example Groups

inner group可以看做是outer group的一个子类  
所以在outer group里声明的任何helper method, before/after  
在inner group里都依然可用

nested example group里的代码执行顺序:
>
  1. Outer before
  2. Inner before
  3. Example
  4. Inner after
  5. Outer after