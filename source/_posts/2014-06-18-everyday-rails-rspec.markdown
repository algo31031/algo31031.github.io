---
layout: post
title: "Everyday Rails RSpec"
date: 2014-06-19 16:20:03 +0800
comments: true
categories: [test, RSpec]
---
前段时间利用零碎时间走马观花的把<The RSpec Book>的前面部分(part1~part4)过了一遍  
发现自己在测试这方面缺口真是不小  
<The RSpec Book>前半部分偏理论性的东西很多, 啃得很生硬  
于是买了本[Andor Chen译的<使用 RSpec 测试 Rails 程序>](https://leanpub.com/everydayrailsrspec-cn)以作实践  

### RSpec

RSpec 2.99之后版本, 需要稍作变动
在`Gemfile`里加上`gem 'rspec-collection_matchers'`

对于** expectation **, 之前的版本惯用的`should`形式  
被改写为`expect(<obj>).to <matcher>`这种用法

** subject() **

``` ruby
describe CheckingAccount, "with $50" do
  subject { CheckingAccount.new(Money.new(50, :USD)) }
  it { is_expected.to have_a_balance_of(Money.new(50, :USD)) }
  it { is_expected.not_to be_overdrawn }
end

describe CheckingAccount, "with a non-zero starting balance" do
  subject(:account) { CheckingAccount.new(Money.new(50, :USD)) }
  it { is_expected.not_to be_overdrawn }
  it "has a balance equal to the starting balance" do
    #account.balance.should eq(Money.new(50, :USD))
    expect(:account).to eq(Money.new(50, :USD))
  end
end
```

** mock & stub **

RSpec 2.14之前的语法要这么写
``` ruby
d = double(:message1 => true)
d.stub(:message2).and_return(:value)
real_object.stub(:message).and_return(:value)
```

之后可以这么写
``` ruby
d = double(:message1 => true)
allow(d).to receive(:message2).and_return(:value)
allow(real_object).to receive(:message).and_return(:value)
```

### FactoryGirl

简化factory_girl代码
``` ruby 加到rails_helper.rb
config.include FactoryGirl::Syntax::Methods
```

** Aliases **

可以用`aliases`简化关联关系
``` ruby
FactoryGirl.define do

  factory :user, aliases: [:author, :commenter] do
    first_name "John"
    last_name "Deo"
  end

  factory :post do
    author  #用以替代 association :author, factory: :user
  end

  factory :comment do
    commerter  #替代  association :commenter, factory: :user
  end

end
```

** Associations **

如果association的名字与factory的名字相同, 可以省略`association`  
也可以指定不同名的factory并覆盖某属性
``` ruby
factory :post do
  author factory: :user, last_name: "skywalker"
```

### Capybara

以下方法别名只可用在功能测试中
>
  `given`对应`let`  
  `background`对应`before`  
  `feature`对应`describe`  
  `scenario`对应`it`

需要注意`feature`块不可嵌套

** 涉及javascript **
``` ruby
feature "About BigCo modal" do
  scenario "toggles display of the modal about display", js: true do
    # blablalbla
```

另外如果开启了`js: true`时, 要测试DB变化, 有可能DB反映慢导致测试失败, 可以采取下面办法:
``` ruby
expect {
  fill_in_new_user_data("new_user@example.com")
  check "user_admin"
  click_button "Create User"
  sleep 1 
}.to change(User, :count).by(1)     
```

### Launchy

`save_and_open_page`

### Guard

`bundle exec guard init rspec`