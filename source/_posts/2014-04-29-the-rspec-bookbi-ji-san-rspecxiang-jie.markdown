---
layout: post
title: "The RSpec Book笔记(三) RSpec详解(1)"
date: 2014-04-29 16:51:59 +0800
comments: true
categories: [test, RSpec]
---
# Part III RSpec

### Chapter 12 Code Examples

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
在传入`it`的代码块中使用了** expectation ** (`account.balance.should == Money.new(0, :USD)`)

** describe 方法 **

参数有以下四种形式:  
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

参数形似与 describe 方法相同   
其字符串参数为以'it'开头的一句话, 用来描述代码块中代码的细节  