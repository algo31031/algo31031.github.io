---
layout: post
title: "使用cucumber配合pickle"
date: 2011-01-27 13:58:00 15:43:10 +0800
comments: true
categories: [测试 cucumber]
---
建立项目  

    $ rails -d mysql order test

添加脚手架 

    $ ruby script/generate scaffold books title:string

创建数据库，建表

修改`BooksController#index`

    @books = Book.find(:all,:order => "title DESC")

在项目下加载cucumber与capybara
    ruby script/generate cucumber --capybara

添加pickle(path可选，加上之后会在paths.rb里面增加)
    ruby script/generate pickle path    

在相应books_steps.rb文件里加入

    Then /^我应该(?:在"([^"]*)"中)?依次看到:$/ do |selector, table|
      pattern = table.raw.flatten.collect(&Regexp.method(:quote)).join('.*?')
      regexp = Regexp.compile(pattern, Regexp::MULTILINE)
      with_scope(selector) do
        page.should have_xpath('//*', :text => regexp)
      end
    end

在feature中加入

    Scenario: books in order
    Given the following books exist
      | title   | id |
      | aaa  | 1  |
      | bbb  | 2  |
      | ccc   | 3  |
    When I go to books
    Then I should see in that order:
      | ccc    |
      | bbb    | 
      | aaa    | 
    And I should see "aaa" 

在paths.rb中加入

    when /books/
      books_path

运行

    $ rake cucumber:all

等着看黄瓜变绿～～