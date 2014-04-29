---
layout: post
title: "The RSpec Book笔记(二) 初窥cucumber与RSpec "
date: 2014-04-29 10:38:36 +0800
comments: true
categories: [test, RSpec, cucumber]
---
### Describing Features

将特性(features)整理成若干用户故事(User Stories), 可以采用`role + action`的形式作为故事的标题.  
故事里不需要包含太多细节, 详细内容可以在选定好选取哪些故事用作发布以及哪次迭代时发布后, 再行考虑.
> * ** Code-breaker starts game ** The code-breaker opens a shell, types a
command, and sees a welcome message and a prompt to enter
the first guess.  
> * ** Code-breaker submits guess ** The code-breaker enters a guess, and
the system replies by marking the guess according to the marking
algorithm.

关于User Stroies, 可以看下iHover大大的[User Stories (1) 什麼是 User Story?](http://ihower.tw/blog/archives/2090)

User Story需要拥有以下特性:
> * ** Have business value **   
> * ** Be testable **   
> * ** Be small enough to implement in one iteration ** 

在项目里添加一个`features`目录,然后在`teatures`下添加`support`目录,  
在`support`目录里添加`env.rb`文件(其实*.rb便可), 这样cucumber会知道我们正在用ruby 

在里`features`面创建一个`codebreaker_submits_guess.feature`文件
{% codeblock lang:cucumber Scenario Example - codebreaker_submits_guess.feature %}
Feature: code-breaker submits guess
  As a code-breaker
  I want to submit a guess
  So that I can try to break the code

  Scenario: all exact matches
    Given the secret code is "1234"
    When I guess "1234"
    Then the mark should be "++++"
{% endcodeblock %}

{% codeblock lang:cucumber Scenario Outline Example - codebreaker_submits_guess.feature %}
Feature: code-breaker submits guess
  As a code-breaker
  I want to submit a guess
  So that I can try to break the code

  Scenario Outline: submit guess
    Given the secret code is "<code>"
    When I guess "<guess>"
    Then the mark should be "<mark>"

  Scenarios: all numbers correct
    | code | guess | mark |
    | 1234 | 1234  | ++++ |
    | 1234 | 1243  | ++-- |
    | 1234 | 1423  | +--- |
    | 1234 | 4321  | ---- | 
{% endcodeblock %}

### Automating Features with Cucumber

在`features`目录下创建`step_definitions`目录, 然后再里面添加一个`codebreaker_steps.rb`文件

** Step Definition Methods **  
> * ** Given() ** 给出背景条件(context)　　
> * ** When() ** 执行动作　　
> * ** Then() ** 校验结果　　
> * ** And()与But() ** 与上一个Given(),When(),或Then()意义相同, 只为使整个描述看起来更近似自然语言.

### Describing Code with RSpec

项目下创建`spec/codebreaker/`, 然后在里面添加`game_spec.rb`  
原则是每个source文件要对应一个spec文件  
``` ruby game_spec.rb
require 'spec_helper'

module Codebreaker
  describe Game do 
    describe "#start" do 
      it "sends a welcome message"
      it "prompts for the first guess"
    end
  end
end
```

在`spec/codebreaker/`下添加一个`spec_helper.rb`
``` ruby spec_helper.rb`
require 'codebreaker'
``` 

it()方法如果不传入代码块, 会被当做pending的方法

可以用`double("xxx")`方法得到一个[test double(测试替身)](http://en.wikipedia.org/wiki/Test_double)  
`double("xxx").as_null_object`会让替身只关心指定给它的被期待的消息, 而忽略其他消息

``` ruby game_spec.rb
require 'spec_helper'

module Codebreaker
  describe Game do
    describe "#start" do
      it "sends a welcome message" do
        output = double('output').as_null_object
        game = Game.new(output)

        output.should_receive(:puts).with('Welcome to Codebreaker!')

        game.start
      end

      it "prompts for the first guess" do
        output = double('output').as_null_object
        game = Game.new(output)

        output.should_receive(:puts).with('Enter guess:')

        game.start
      end
    end
  end
end
```

** before(:each) {} **  
传入block里的内容在每个example的顶部执行  
可用这个方法创建实例变量并赋值
``` ruby
before(:each) do
  @output = double('output').as_null_object
  @game = Game.new(@output)
end
```


** let(:method) {} **  
传入的symbol作为方法名, 传入的block被当做方法体
``` ruby
let(:output) { double('output').as_null_object }
let(:game) { Game.new(output) }
```
