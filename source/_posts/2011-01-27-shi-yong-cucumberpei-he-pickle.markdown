---
layout: post
title: "使用cucumber配合pickle"
date: 2011-01-27 13:58:00 15:43:10 +0800
comments: true
categories: [测试 cucumber]
---
建立项目  

{% codeblock lang:bash %}
$ rails -d mysql order test
{% endcodeblock %}

添加脚手架 

{% codeblock lang:bash %}
$ ruby script/generate scaffold books title:string
{% endcodeblock %}


创建数据库，建表

修改`BooksController#index`

{% codeblock lang:ruby BooksController#index %}
@books = Book.find(:all,:order => "title DESC")
{% endcodeblock %}

在项目下加载cucumber与capybara

{% codeblock lang:bash %}
$ ruby script/generate cucumber --capybara
{% endcodeblock %}

添加pickle(path可选，加上之后会在paths.rb里面增加)

{% codeblock lang:bash %}
$ ruby script/generate pickle path    
{% endcodeblock %}

在相应books_steps.rb文件里加入

{% codeblock lang:ruby books_steps.rb %}
Then /^我应该(?:在"([^"]*)"中)?依次看到:$/ do |selector, table|
  pattern = table.raw.flatten.collect(&Regexp.method(:quote)).join('.*?')
  regexp = Regexp.compile(pattern, Regexp::MULTILINE)
  with_scope(selector) do
    page.should have_xpath('//*', :text => regexp)
  end
end
{% endcodeblock %}


在feature中加入

{% codeblock lang:cucumber  %}
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
{% endcodeblock %}

在paths.rb中加入

{% codeblock lang:ruby  path.rb %}
when /books/
  books_path
{% endcodeblock %}

运行
{% codeblock lang:bash %}
$ rake cucumber:all
{% endcodeblock %}

等着看黄瓜变绿～～