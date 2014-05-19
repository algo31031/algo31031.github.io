---
layout: post
title: "The RSpec Book笔记(六) RSpec::Mocks"
date: 2014-05-19 16:04:29 +0800
comments: true
categories: [test, RSpec]
---
## Chapter 14 RSpec::Mocks

### Test Doubles, Method Stubs, Message Expectations

``` ruby
thingamajig_double = double('thing-a-ma-jig')
stub_thingamajig = stub('thing-a-ma-jig')
mock_thingamajig = mock('thing-a-ma-jig')
```

`double()`,`stub()`,`mock()`都会返回一个`RSpec::Mocks::Mock`的实例  
可以在这个实例上生成method stubs和message expectations  

``` ruby
describe Statement do
  it "logs a message on generate()" do
    customer = stub('customer')
    customer.stub(:name).and_return('Aslak')

    logger = mock('logger')

    statement = Statement.new(customer, logger)
    logger.should_receive(:log).with(/Statement generated for Aslak/)

    statement.generate
  end
end
```

这段代码中, `stub('customer')`和m`mock('logger')`分别生成了2个test double

`customer.stub(:name)`为customer double添加了一个method stub(打桩方法), :name为方法名
`and_return('Aslak')`表示:name的返回值为Aslak

`logger.should_receive(:log)`为logger double设置了一个对于message `name()` 的expectation  
后面的`generate()`方法里, 如果`logger`对`log()`的调用失败, 则整个example会fail  
否则会判断`logger.should_receive(:log)`后面的条件是否满足(此处为`with(xxx)`,即`log()`调用是否带参数xxx)

** partial stubbing, partial mocking **

``` ruby
describe WidgetsController do
  describe "PUT update with valid attributes"
    it "finds the widget"
      widget = Widget.new()
      widget.stub(:update_attributes).and_return(true)
      Widget.should_receive(:find).with("37").and_return(widget)
      put :update, :id => 37
    end

    it "updates the widget's attributes" do
      widget = Widget.new()
      Widget.stub(:find).and_return(widget)
      widget.should_receive(:update_attributes).and_return(true)
      put :update, :id => 37
    end

    it "redirects to the list of widgets"
      widget = Widget.new()
      Widget.stub(:find).and_return(widget)
      widget.stub(:update_attributes).and_return(true)
      put :update, :id => 37
      response.should redirect_to(widgets_path)
    end    
  end
end
```

### 更多关于Method Stubs

** One-Line Shortcut **

`double()`,`stub()`,`mock()`第一个参数非必填但是强烈建议有, 因为其会作为失败时的消息  
此外还可以接受一个hash作为第二参数
``` ruby
customer = double('customer', :name => 'Bryan')
```

等效于
``` ruby
customer = double('customer')
customer.stub(:name).and_return('Bryan')
```

** Implementation Injection **

如果一个stub method需要使用多次而且根据条件不同会有不同返回值, 可以用如下方法  
多用于`before()`里
``` ruby 
ages = double('ages')
ages.stub(:age_for) do |what|
  if what == 'drinking'
    21
  elsif what == 'voting'
    18
  end
end
```

** 方法链 **

``` ruby 
article = double()
Article.stub_chain(:recent, :published, :authored_by).and_return(article)
```

### 更多关于Message Expectations

** 执行次数 **

`should_receive(:xxx)`要求`xxx()`被调用且只调用一次, 如果希望调用若干次, 可采用下列方式

``` ruby
mock_account.should_receive(:withdraw).exactly(5).times
network_double.should_receive(:open_connection).at_most(5).times
network_double.should_receive(:open_connection).at_least(2).times
account_double.should_receive(:withdraw).once
account_double.should_receive(:deposit).twice
```

如果期待方法不被调用,要使用`should_not_receive`

``` ruby
network_double.should_not_receive(:open_connection)
network_double.should_receive(:open_connection).never              #不推荐
network_double.should_receive(:open_connection).exactly(0).times   #不推荐
```

** 指定期待的参数, `with()` **

``` ruby
### 指定一个参数, 且值为50
account_double.should_receive(:withdraw).with(50)
### 可以指定任意个参数
checking_account.should_receive(:transfer).with(50, savings_account)
### 第一个参数为给定值, 第二个参数为Fixnum型任意值
source_account.should_receive(:transfer).with(target_account, instance_of(Fixnum))
### 第一个参数可以为任意类型任意值
source_account.should_receive(:transfer).with(anything(), 50)
### 任意类型任意数量参数
source_account.should_receive(:transfer).with(any_args())
### 不传参数
collaborator.should_receive(:message).with(no_args())
### 参数为包含/不包含给定key/value的Hash
with(hash_including('Electric' => '123', 'Gas' => '234'))
with(hash_not_including('Electric' => '123', 'Gas' => '234'))
### 正则表达式
mock_atm.should_receive(:login).with(/.* User/)
```

** 自定义的Argument Matchers **

自定义一个类, 然后重写`==(actual)`方法即可  
也可以添加一个`description()`方法以提供失败时输出的信息

``` ruby
class GreaterThanMatcher

  def initialize(expected)
    @expected = expected
  end

  def description
    "a number greater than #{@expected}"
  end

  def ==(actual)
    actual > @expected
  end
end

def greater_than(floor)
  GreaterThanMatcher.new(floor)
end

calculator.should_receive(:add).with(greater_than(37))

```

** Throwing or Raising **

`and_raise()`可以不传参/一个参数(异常类或异常类实例)  
`and_throw()`传symbol

``` ruby
account_double.should_receive(:withdraw).and_raise
account_double.should_receive(:withdraw).and_raise(InsufficientFunds)

the_exception = InsufficientFunds.new(:reason => :on_hold)
account_double.should_receive(:withdraw).and_raise(the_exception)

account_double.should_receive(:withdraw).and_throw(:insufficient_funds)
```

** 按序执行 **

``` ruby
database.should_receive(:count).with('Roster', :course_id => 37).ordered
database.should_receive(:add).with(student).ordered
```
只要`count()`在`add()`之前执行就会pass