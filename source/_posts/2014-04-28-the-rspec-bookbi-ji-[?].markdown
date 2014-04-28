---
layout: post
title: "The RSpec Book笔记(一) Part I  Getting Started"
date: 2014-04-28 10:58:54 +0800
comments: true
categories: [test, RSpec, cucumber]
---
其实这个坑经开了有些日子, 测试一直都是自己弱项.  
最初在云清扬时就很少写, 后来开始搞爱豆网人力有限加之是摸索中需求不断变化的初创, 索性一行测试代码都没有.  
现在在做对日外包, 日方那里直接要求的就是完全肉测然后上测试式样书, 更是没机会写测试.  
适逢DHH大神前不久的[TDD is dead. Long live testing.](http://david.heinemeierhansson.com/2014/tdd-is-dead-long-live-testing.html), 引发关于测试的各种大讨论

但是无论大神观点如何, 毕竟自己只是一枚小小的程序猿, 而TDD作为一种很成熟开发方式, 在很多情况下依然会是行之有效的. 我一直信奉所谓"存在即合理".

碰巧这段时间开发进度不是很忙, 又快赶上5.1的三天假期, 应该可以把手头的<The RSpec Book>啃掉. 然后把chat_demo 改以书中所倡导的BDD的方式改善下, 通过实践来加深理解.

# Part I Getting Started with RSpec and Cucumber

### 什么是TDD
TDD(Test-Driven Development)这个词并不陌生, 曾经开发亲情网时整个team也煞有介事的有过一小段时间的BDD尝试.    
但是归根结底, TDD其重点应该落在Development上, 是一种包含需求分析,设计,测试,编码于一体的开发方法, 而并不仅仅是一种写Test的手段.  

TDD要求我们在开发时先写出一个简单的测试, 这时运行测试一定是无法通过的, 因为还没开始编码; 然后编写最低限度的代码, 使测试通过.  
一旦测试通过后, 需要重新审视我们的设计并重构代码去除冗余. 此时我们手头的代码, 毫无疑问地太过简单而无法处理全部需求.  
相对于直接添加代码, 我们此时该做的是在测试里增加新的特性让测试失败, 然后再编写最低限度通过测试的代码, 回顾设计, 重构...  
如此反复, 直到我们完成整个功能.  

整个这个循环往复的过程, 又被称为红绿重构(red/green/refactor).

有些时候, 我们即是开发者又是测试者. 如果遇到这种情况, 把test与TDD的情境区分开依然是有帮助的: 作为TDDer时, 把注意力集中在红绿重构,设计,规范要求上; 而作为tester时, 则需要尽可能考诸如虑如何设置边界条件,如何发掘隐藏的bug等等.

### 那么BDD又是什么
我们测试一个对象内部结构的问题在于: 我们只是测试了这个对象是什么,而没有关心它可以做些什么. 而后者无疑远比前者更为重要.

BDD(Behaviour-Driven Development)则把目光着眼于行为(做什么)而非结构(是什么).  
它将程序以更近似自然语言的方式, 描述为一个个场景(scenario): ** Given ** some context, ** When ** some event occurs,
** Then ** I expect some outcome. 这样做可以大幅降低沟通成本.

通过Given, When, Then三位一体的方式, 可以很容易的描述出程序的行为以及对象的行为. 并且这种描述, 分析人员,测试人员,卡发人员都能很好地理解.

### BDD需要些什么

``` ruby RSpec使用
rspec [options] [files or directories]
```

``` ruby cucumber使用
cucumber [options] [ [FILE|DIR|URL][:LINE[:LINE]*] ]+
```

在rails项目里添加RSpec和cucumber

``` ruby 添加进Gemfile  
group :development, :test do
  gem 'rspec'
  gem 'rspec-rails'
  gem 'cucumber'
  gem 'cucumber-rails'
  gem 'database_cleaner'
  gem 'webrat'
  gem 'selenium-client'
end
```
运行`script/rails generate rspec:install`  
会在项目跟目录下生成 `spec/spec_helper.rb` 与 `.rspec`   
`.rspec`文件为RSpec的配置文件, 可以放在项目根目录下, 或放在主目录/home/xxx/下

``` bash .rspec
--color 
--format doc
--backtrace
```

# Part II Behaviour-Driven Development

### The Principles of BDD

> ** Enough is enough ** 不要过度设计  
> ** Deliver stakeholder value ** 不做不相关的事  
> ** It’s all behavior ** RSpec描述程序行为, cucumber描述用户行为  

***

### 三段式的故事描述
> as a [stakeholder],  
> I want [feature]  
> so that [benefit].  

或者:  

> in order to [benefit],  
> a [stakeholder]  
> wants to [feature].  

更推荐后者，因为其更突出行为的目的。

