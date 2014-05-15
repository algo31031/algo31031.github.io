---
layout: post
title: "The RSpec Book笔记(五) RSpec::Expectations"
date: 2014-05-15 15:17:39 +0800
comments: true
categories: [test, RSpec]
---
## Chapter 13 RSpec::Expectations

** assertions OR expectations **  
BDD里, 使用expectations替代了传统测试里的assertions  
虽然作用基本是一样的, 但是2者理念不同

传统测试我们先有了代码, 于是我们可以断言(assert)一段代码执行之后会出现生么状况  
但是在BDD中, 测试之前还没有代码本体. 我们把自己化身为各种角色, 做出各种行为, 然后期待(expect)会得到某样结果

### 13.1 should, should_not, and matchers

RSpec为所有Ruby对象添加了`should()`和`should_not()`方法  
每个方法可以接受一个`matcher`对象或一个包含了特定范围内的ruby操作符的ruby表达式作参数

``` ruby
result.should equal(5)
```

这段代码会先对`equal(5)`求值, 这是一个RSpec提供的方法, 可以返回一个matcher对象,  
之后这个matcherd对象被传给`result.should`.  
`should`会调用`matcher.matches?`, 并把`self`(在这里即是`result`)作为参数.  
如果`matches?(self)`返回`true`, 则expectations通过, 开始执行example里下一行代码,  
否则`should()`方法会向matcher索取错误信息并报`ExpectationNotMetError`

### 13.2 Built-in Matchers

``` ruby
include(item)
respond_to(message)
raise_error(type)
```

** 相等 **

对于[ruby中的四种相等](http://blog.banban.me/blog/2014/05/14/rubyxia-de-equals-equals-equals/), rspec提供了以下四种对应:
``` ruby
a.should == b
a.should === b
a.should eql(b)
a.should equal(b)
```
另外还要注意** 不要使用 `!=` **, 而是要用RSpec里提供的`should_not`方法  
因为`==`实际是调用的ruby方法, 于是
`actual.should == expected`会被解释称`actual.shoul. ==(expected)`

而`actual.should != expected`会被解释成`!(actual.should.==(expected))`  
actual和expected如果不等, 则会直接报`ExpectationNotMetError`了

** 浮点数 ** 

``` ruby
result.should be_close(5.25, 0.005)
```

** 多行文本 **

``` ruby
result.should match(/this expression/)
result.should =~ /this expression/
```

** 变更 **

``` ruby
expect {
  User.create!(:role => "admin")
}.to change{ User.admins.count }

expect {
  User.create!(:role => "admin")
}.to change{ User.admins.count }.by(1)

expect {
  User.create!(:role => "admin")
}.to change{ User.admins.count }.to(1)

expect {
  User.create!(:role => "admin")
}.to change{ User.admins.count }.from(0).to(1)

expect {
  seller.accept Offer.new(250_000)
}.to change{agent.commission}.by(7_500)
```

最后面那个也可以写成
``` ruby
agent.commission.should == 0
seller.accept Offer.new(250_000)
agent.commission.should == 7_500
```

** 报错 **

``` ruby
expect {
  account.withdraw 75, :dollars
}.to raise_error(
  InsufficientFundsError,
  /attempted to withdraw 75 dollars/
)
```

`raise_error`可以传0个,1个或2个参数  
第一个参数可以是error class, 错误信息的字符串 或者匹配错误信息的正则表达式  
如果第一个参数为error class, 可以传第二个参数, String或Regexp

** 手动抛出异常 **

``` ruby
course = Course.new(:seats => 20)
  20.times { course.register Student.new }
lambda {
  course.register Student.new
}.should throw_symbol(:course_full, 20)
```

参数形式与`raise_error`类似, 但是第一个参数必须为symbol, 第二个参数可为任意类型

### 13.3 Predicate Matchers

所谓predicate method, 即是以`?`并且返回一个`boolean`值的方法  
在RSpec里, 可以用`be_xxx`, `be_a_xxx`, `be_an_xxx` 来描述一个predicate method

### 13.4 Be True in the Eyes of Ruby

``` ruby
true.should be_true
0.should be_true
"this".should be_true

false.should be_false
nil.should be_false
```

对于特殊的只期待true/false的场合, 可以使用`equal`

``` ruby
true.should equal(true)
false.should equal(false)
```

### 13.5 Have Whatever You Like

** have_xxx **

对`has_xxx?`这类predicate method, 可以使用`have_xxx`的Predicate Matchers

``` ruby
request_parameters.has_key?(:id).should == true
request_parameters.should have_key(:id)
```

** Owned Collections **

``` ruby 
field.players.select {|p| p.team == home_team }.length.should == 9
```

够ruby, 但是不够English, 于是可以写成

``` ruby
home_team.should have(9).players_on(field)
```

其中, `have()`返回一个无法响应`players_on()`方法的matcher  
之后这个matcher把`players_on()`方法代理到`home_team`上

这么写以来易读(从English角度), 二来可以鼓励添加诸如`players_on()`这样的有用的方法

** Unowned Collections **

对于需要描述的对象本身就是collection的情况, 需要判断其size/length

``` ruby
collection.should have(37).items
```

其中`items`是语法糖, 后面会有进一步说明

同样的, String也适用, 其中`characters`也是语法糖

``` ruby
"this string".should have(11).characters
```

除了`have()`以外, 还有`have_at_least()`和`have_at_most()`  
`have_exactly()`与`have()`同义

``` ruby
day.should have_exactly(24).hours
dozen_bagels.should have_at_least(12).bagels
internet.should have_at_most(2037).killer_social_networking_apps
```

** have()究竟如何工作的 **

`have()`方法会返回一个`RSpec::Matchers::Have`实例  
实例里面记录了传入`have()`的数目作为指定collection的包含元素数目的期待值

``` ruby
result.should have(3).things
```

其实等价于

``` ruby
result.should(Have.new(3).things)
```

Have类重写了`method_missing`方法, 使其能记录自己无法响应的方法(在这里就是`things`), 并返回self  
于是`Have.new(3).things`, 最终返回了一个包含期待的collection元素数目(3)以及可能的collection名字(`things`)的Have对象

紧接着, 这个Have对象被传递给了`should()`方法  
`should()`调用`matcher.matches?(self)`

而在`matches?()`方法里, 首先会判断目标对象(`result`)是否能响应之前纪录的`things`方法  
若能相应, 则在`result.things`上调用`length`或者`size`(length优先)  
此时如果`result.things`没有`length`或`size`, 就会得到一个error message
如果有`length`/`size`, 便会与Have对象里记录的数目作比较, 判断example通过或者失败

如果`result`无法响应`things`, 则会在`result`自身调用`length`或`size`  
之后的判断与上面一样,  返回错误信息或者比较数目是否相等 

### 13.6 Operator Expressions

``` ruby
result.should == 3
result.should =~ /some regexp/
result.should be < 7
result.should be <= 7
result.should be >= 7
result.should be > 7
```

这些会被Ruby解释成

``` ruby
result.should.==(3)
result.should.=~(/some regexp/)
result.should(be.<(7))
result.should(be.<=(7))
result.should(be.>=(7))
result.should(be.>(7))
```

RSpec在`should()`返回的对象上定义了`==`, `=~`, 在`be()`返回对象上定义了`<`, `<=`, `>=`, `>`

### 13.7 Generated Descriptions

``` ruby
describe "A new chess board" do
  before(:each) do
    @board = Chess::Board.new
  end

  it "should have 32 pieces" do
    @board.should have(32).pieces
  end
end
```
因为example运行时输出的内容几乎和example是一样的, 于是上述内容也可以写成

``` ruby
describe "A new chess board" do
  before(:each) { @board = Chess::Board.new }
  specify { @board.should have(32).pieces }
end
```  
这2段代码输出的内容都是
``` bash
A new chess board
  should have 32 pieces
```

`specify()`和`it()`一样, 都是`example()`的方法别名

### 13.8 Subjectivity

`subject()`相当于在`before`里创建一个当前example的subject(被描述的对象)

``` ruby
describe Person do
  subject { Person.new(:birthdate => 19.years.ago) }
  specify { subject.should be_eligible_to_vote }
end
```

一旦`subject`被声明了, `should()`和`should_not()`都会被代理到`subject`上  
于是上面的代码还可以写作
``` ruby
describe Person do
  subject { Person.new(:birthdate => 19.years.ago) }
  it { should be_eligible_to_vote }
end
```

如果创建的subject在执行`new`时不需要参数, `subject`的声明也可以不写, 直接写作

``` ruby
describe RSpecUser do
  it { should be_happy }
end
```