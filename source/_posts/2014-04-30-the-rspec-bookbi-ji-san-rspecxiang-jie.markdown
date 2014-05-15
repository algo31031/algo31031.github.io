---
layout: post
title: "The RSpec Book笔记(三) Code Examples(1)"
date: 2014-04-30 11:55:59 +0800
comments: true
categories: [test, RSpec]
---
# Part III RSpec

## Chapter 12 Code Examples

### 12.1 Describe It!

** 名词解释 **  
> * ** subject code **  以RSpec描述出其行为的代码  
> * ** expectation **   等同于Assertion(断言)  
> * ** code example **  等同于Test method,用来展示** subject code **的作用,并通过** expectation **表现其行为  
> * ** example group ** 等同于Test case, 一组** code example **  
> * ** spec **          spec文件, 包含一个或多个** example group **  

例如:  
``` ruby
describe "A new Account" do
  it "should have a balance of 0" do
    account = Account.new
    account.balance.should == Money.new(0, :USD)
  end
end
```    

`describe`方法定义了一个** example group **  
`describe`传入的字符串代表我们要描述的系统的[facet](http://www.iciba.com/facet)(一个新账户)   
`it`方法定义** code example **, 传入的字符串用来描述我们所关心的facet的行为(余额应为0)  
在传入`it`的block中使用了** expectation ** (`account.balance.should == Money.new(0, :USD)`)

** describe 方法 **

参数有以下几种形式:  
``` ruby 
describe "A User" { ... }  
=> A User  

describe User { ... }  
=> User  

describe User, "with no roles assigned" { ... }  
=> User with no roles assigned  

describe User, "should require password length between 5 and 40" { ... }  
=> User should require password length between 5 and 40  
```

其中第一个参数可以是class\module或字符串  
若为class\module并且ExampleGroup被包含在module里  
则output会连module名一起输出  
``` ruby
module Authentication
  describe User, "with no roles assigned" do
```
会输出`Authentication::User with no roles assigned`

第二个参数为字符串, 可不填

我们也可以嵌套example groups 

例如:  
``` ruby
describe User do
  describe "with no roles assigned" do
    it "is not allowed to view protected content" do
```
会输出:  
``` 
User
  with no roles assigned
    is not allowed to view protected content
```    

** context方法 **

context是describe方法的别名,一般倾向于用describe描述事物, 用context表述背景(条件)   
于是上面的例子一般来说会写作:  
``` ruby
describe User do
  context "with no roles assigned" do
    it "is not allowed to view protected content" do
```    

** it 方法 **

与describe方法类似, it可接收的参数包括一个单独的字符串,一个可选的hash和一个可选的block   
其字符串参数若为以'it'开头的一句话, 则代表这句话的详情会通过block里的代码表示出来.

例子:
``` ruby
describe Stack do
  before(:each) do
    @stack = Stack.new
    @stack.push :item
  end

  describe "#peek" do
    it "should return the top element" do
      @stack.peek.should == :item
    end

    it "should not remove the top element" do
      @stack.peek
      @stack.size.should == 1
    end
  end

  describe "#pop" do
    it "should return the top element" do
      @stack.pop.should == :item
    end

    it "should remove the top element" do
      @stack.pop
      @stack.size.should == 0
    end
  end
end
```
以上代码通过example group的嵌套, 将`peak()`和`pop()`2个group分隔开  
如果运行时加上`--format documentation`参数的, 会得到如下输出:
``` bash
Stack
  #peek
    should return the top element
    should not remove the top element
  #pop
    should return the top element
    should remove the top element

Finished in 0.00154 seconds
4 examples, 0 failures
```

### 12.2 Pending Examples  

1.  ** 用于代码还未实现的 **

    在`it`方法里不传代码块, 则此code example会被当做pending的example

        describe Newspaper do
          it "should be read all over"
        end

    运行RSpec时, 会在output里有:

        Newspaper
          should be read all over (PENDING: Not Yet Implemented)  

        Pending:
          Newspaper should be read all over
            # Not Yet Implemented
            # ./newspaper_spec.rb:17

    可用于列出还未实现的功能  

2.  ** 用于已有代码但是需要修改的 **

    加入pending声明, 不追加代码块, 则声明后的代码不会被执行  

        describe "onion rings" do
          it "should not be mixed with french fries" do
            pending "cleaning out the fryer"
            fryer_with(:onion_rings).should_not include(:french_fry)
          end
        end

    可以使测试依然通过而不必将原先内容注释掉  

3.  ** 用于bug report **

    如果此bug当前并不想修改  
    可以把有问题的代码置于pending下, 避免其被执行(类似上面2.的情形)  

    传入pending的代码块会被执行, 若其不通过或者报错, 则会像普通pending  
    否则RSpec会报`PendingExampleFixedError`, 提醒你此处无缘无故pending了  
    然后即可将pending移除, 因为这些代码已经可以通过测试

        describe "an empty array" do
          it "should be empty" do
            pending("bug report 18976") do
              [].should be_empty
            end
          end
        end

    以上代码执行后会有如下输出:

        F

        Failures:
          1) an empty array should be empty FIXED
            Expected pending 'bug report 18976' to fail. No Error was raised.
            # ./pending_fixed.rb:4
    
        Finished in 0.00088 seconds
        1 example, 1 failure


### 12.3 Hooks: Before, After, and Around

1.  ** before(:each) **  
    对于example group中的每个example, 都会重新运行一遍  

2.  ** before(:all) **  
    只在自身对象的实例中运行一次(This gets run once and only once in its own instance of Object),  
    但在其中的实例变量会被copy到每个example下   

    需要注意before(:all)有可能造成不同group之间的状态共享,  
    所以除非特殊情况(如需打开网络连接), 尽量都用before(:each)

3.  ** after(:each) **  
    在其中的代码一定会执行, 即使examples或着其他hooks里的代码无法通过甚至报错  
    after(:each)可用于恢复全局变量状态  

        before(:each) do
          @original_global_value = $some_global_value
          $some_global_value = temporary_value
        end

        after(:each) do
          $some_global_value = @original_global_value
        end

4.  ** after(:all) **  
    不常用, 可用于最后关闭诸如DB连接等  

5.  ** around(:each) **  
    会把当前运行的example当做代码块传入around, 然后可运行example.run  
    可用于database transactions:

        around do |example|
          DB.transaction { example.run }
        end
    
    也可以把example作为代码块直接传给around里的方法, 如:

        around do |example|
          ansaction &example
        end

    除此之外也可以用于类似下面这种情况:

        around do |example|
          begin
            # do something
            example.run
          ensure 
            # do something else
          end
        end

    但是上面这种情况会降低代码可读性, 所以如遇以上情况还是使用before/after为好:

        before { do_some_stuff_before }
        after { do_some_stuff_after } 