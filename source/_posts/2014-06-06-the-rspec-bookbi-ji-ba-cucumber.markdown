---
layout: post
title: "The RSpec Book笔记(八) cucumber"
date: 2014-06-09 10:11:16 +0800
comments: true
categories: [test, cucumber]
---
### Given/When/Then

** Given ** 用来表示在一个scenario中我们认可为true的事物.  
通过这个声明来给出在scenario中要发生的事件的上下文语境  

given经常被误认为是先决条件, 但两者有概念上的不同:  
先决条件是指某种强制约束, 如果达不到某一条件就无法继续进行下去;  
但是given并被没有这种强制性, 为了使特定行为能满足其所需条件, given给出的条件可以被打破

换言之, 可以有`Given the world is round`, 但绝不会有`Given the world is flat`

** When ** 用来表示scenario中的事件, 倾向于每个scenario只有一个独立事件

** Then ** 表示期待的结果

### Tags

通过类似实例变量`@xxx`的形式指定tag,可以对feature和scenario指定任意数量的tag

``` cucumber
@approved @iteration_12
Feature: patient requests appointment

  @wip
  Scenario: patient selects available time
```

scenario会继承feature的tag, 运行时通过`--tags`指定执行

``` bash
cucumber --tags @wip             #执行全部有@wip的scenarios
cucumber --tags @foo,@bar        #执行有@foo或者@bar的scenarios
cucumber --tags @foo --tags @bar #执行同时有@foo和@bar的
cucumber --tags ~@dev            #没有@dev的
```

### 参数
在step definations中的正则表达式如果包含caputre group, 则会把他们作为参数传给代码块, 例如
``` ruby
Given /^a hotel with "([^"]*)" rooms and "([^"] *)" bookings$/ do 
  |room_count, booking_count|
  # blablabla
end
```

而这些step使用时候, 参数的部分也最好用引号括起来(非强制)
``` ruby
Scenario: Successful booking
  Given a hotel with "5" rooms and "0" bookings
```

### World
所有的cucumber scenario都运行在一个被称作World的对象新的实例上下文里  
默认情况下World只是Object的实例, 在每个scenario之前被实例化  
对于同一scenario的所有step definitions, 其代码块都在相同的上下文执行

可以使用`World()`方法自定义World, 方法接受一个或多个module
``` ruby
module MyHelper
  def some_helper
    ...
  end
end

World(MyHelper)
```

可以在features或其子目录下的任意ruby文件里配置自定义的World,  
但是推荐的做法是将其放在`features/support/world.rb`下

除了可以在World里混入代码块之外, 还可以更改用于实例化World的class的类型,  
只需要在`World()`方法传入代码块
``` ruby
class MyWorld
  def some_helper
    ...
  end
end

World do
  MyWorld.new
end
```

### Calling Steps Within Step Definitions
``` ruby
When /I transfer (.*) from (.*) to (.*)/ do |amount, source, target|
  When "I select #{source} as the source account"
  When "I select #{target} as the target account"
  When "I set #{amount} as the amount"
  When "I click transfer"
end
```

以上代码等效于:

``` ruby
When /I transfer (.*) from (.*) to (.*)/ do |amount, source, target|
  steps %Q{
    When I select #{source} as the source account
    And I select #{target} as the target account
    And I set #{amount} as the amount
    And I click transfer
  }
end
```

### Tagged Hooks
Before("@foo,~@bar", "@zap") do
  puts "This will run before each scenario tagged with @foo or not @bar AND @zap"
end

### Background
有时hooks非技术人员难以理解, 可以用background
``` ruby
Feature: invite friends
  Background: Logged in
    Given I am logged in as "Aslak"
    And the following people exist:
      | name   | friend? |
      | David  | yes     |
      | Vidkun | no      |

  Scenario: Invite someone who is already a friend
  Scenario: Invite someone who is not a friend
  Scenario: Invite someone who doesn't have an account
```

background在给定的feature里每个scenario之前执行,  
如果有`Before` hooks, 则先执行`Before`, 再执行background

### Configuration
可以在为cucumber添加配置文件, 放在`cucumber.yml`或`config/cucumber.yml下`  
``` yaml cucumber.yml
wip: --tags @wip features
```
执行cucumber时
``` bash
cucumber -p wip
```