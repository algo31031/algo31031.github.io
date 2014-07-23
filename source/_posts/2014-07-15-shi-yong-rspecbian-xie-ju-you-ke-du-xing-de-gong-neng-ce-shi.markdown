---
layout: post
title: "使用RSpec编写具有可读性的功能测试(译)"
date: 2014-07-15 10:37:24 +0800
comments: true
categories: [test, RSpec, translation]
---
原文出自[about.futurelearn.com](https://about.futurelearn.com/blog/how-we-write-readable-feature-tests-with-rspec/?utm_source=rubyweekly&utm_medium=email),感谢作者[Chris Zetter](https://www.futurelearn.com/profiles/1390)

**Chris Zetter是FutureLearn产品组的一名开发者，他为我们讲述了自己的小组为了使功能测试兼具可维护性与可读性，在把Cucumber替换为RSpec之后是如何来编写测试的。**

测试是建立与维护一个大型平台不可或缺的一部分。每当我们为FutureLearn这个平台增添新功能时，我们都会编写自动化的功能测试来记录这些新功能是如何运作的，并确保他们不运转时我们也能知晓。

###令人爱恨交加的Cucumber

Cucumber是一款用来编写功能测试的常用工具，每当我们开启项目时它都是我们的不二选择。它可以让我们以用户的视角编写出高层级的行为驱动测试。

``` ruby
Feature: Enrolment
  Scenario: Enrolling in a course
    Given there is a course
    And I am logged in as a learner
    When I enrol on a course
    Then the course should appear in 'my courses'
```

我们乐于使用Cucumber因为它可以使根据用户故事编写测试变得简单易行，而且写完的测试通俗易懂。然而使用Cucumber也有些许不足之处。首先，我们已经在项目里使用了RSpec，再引入Cucumber意味着又要多维护一个测试框架；其次，由于两者的DSLs和测试运行器不同，在他们之间进行脑筋切换又会带来额外开销；最后，我们特别不喜欢Cucumber所使用的正则表达式，因为同Ruby的标准方法调用相比，它们使测试变得更加晦涩难懂。

###编写更好的RSpec features

那么，我么该如何在不失测试可读性的前提下停用Cucumber呢？

我们已经开始使用RSpec features来替代Cucumber，它们通常看起来会是这样：

```ruby
feature 'Enrolment' do
  scenario 'Enrolling in a course' do
    course = FactoryGirl.create(:course)

    learner = FactoryGirl.create(:learner)
    login_as learner

    visit course_path(course)
    find('.join').click
    expect(page).to have_content('Thanks for joining!')

    visit '/'
    expect(page).to have_main_header('My Courses')
    expect(page).to have_content(course.full_title)
  end
end
```

它们总是趋于变得很长，使得难以辨明其究竟在测试些什么。而且难以区分诸如Arrange, Act, Assert（在Cucumber里又被称为'Given'、'When'和'Then'）这些部分。我们试过在代码中这些步骤里添加注释，但它们就和通常那些程序代码里的注释一样不尽如人意：一段时间之后这些注释就变得与实际代码不同步了。

一般来说，如果是在程序里别的地方写出这么长的方法，我们就会有所警觉，并且通常会采用提取方法的办法进行重构。何不也这么做呢？让我们依据Cucumber步骤的风格，把这些代码也提取成一个个方法吧。

```ruby
feature 'Enrolment' do
  scenario 'Enrolling in a course' do
    given_there_is_a_course
    and_i_am_logged_in_as_a_learner
    when_i_enrol_on_a_course
    then_the_course_should_appear_in_my_courses
  end

  def given_there_is_a_course
    @course = FactoryGirl.create(:course)
  end

  def and_i_am_logged_in_as_a_learner
    @learner = FactoryGirl.create(:learner)
    login_as @learner
  end

  def when_i_enrol_on_a_course
    visit course_path(@course)
    find('.join').click
    expect(page).to have_content('Thanks for joining!')
  end

  def then_the_course_should_appear_in_my_courses
    visit '/'
    expect(page).to have_main_header('My Courses')
    expect(page).to have_content(@course.full_title)
  end
end
```

###我们有何发现

我们移除了全部的Cucumber功能测试，并把它们中大部分用新式的RSpec features加以重写。这样一来即可保证拥有Cucumber所提供的优秀的可读性，又使得测试变得更加便于编写和维护。

我们做了一个慎重的决定，不把各个features文件里提取的方法进行复用，因为担心这么做会使得测试难于理解。我们发现在编写一个feature下的多条scenario时，会不自觉的就想要进行代码复用。