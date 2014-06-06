---
layout: post
title: "The RSpec Book笔记(八) cucumber"
date: 2014-06-06 09:11:16 +0800
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