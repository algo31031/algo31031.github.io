---
layout: post
title: "ruby编程风格(节选)"
date: 2014-04-22 14:47:21 +0800
comments: true
categories: ruby
---
平时还是挺注意格式问题的, 不过看来还是有好多地方需要改进  
从ruby-china wiki那边抄录了些平时欠缺的coding style, 日后多加注意

### 源代码布局

*   对于没有内容的类定义，尽可能使用单行类定义形式.
``` ruby 
# bad
class FooError < StandardError
end

# okish
class FooError < StandardError; end

# good
FooError = Class.new(StandardError)
```

* 避免单行方法。即便还是会受到一些人的欢迎，这里还是会有一些古怪的语法用起来很容易犯错.  
    无论如何 - 应该一行不超过一个单行方法.  
    但是**空方法**例外
``` ruby 
# good
def some_method
  body
end

# good
def no_op; end
```

* 当赋值一个条件表达式的结果给一个变量时，保持分支的缩排在同一层。
``` ruby
# bad - pretty convoluted
kind = case year
when 1850..1889 then 'Blues'
when 1890..1909 then 'Ragtime'
when 1910..1929 then 'New Orleans Jazz'
when 1930..1939 then 'Swing'
when 1940..1950 then 'Bebop'
else 'Jazz'
end

result = if some_cond
  calc_something
else
  calc_something_else
end

# good - it's apparent what's going on
kind = case year
       when 1850..1889 then 'Blues'
       when 1890..1909 then 'Ragtime'
       when 1910..1929 then 'New Orleans Jazz'
       when 1930..1939 then 'Swing'
       when 1940..1950 then 'Bebop'
       else 'Jazz'
       end

result = if some_cond
           calc_something
         else
           calc_something_else
         end

# good (and a bit more width efficient)
kind =
  case year
  when 1850..1889 then 'Blues'
  when 1890..1909 then 'Ragtime'
  when 1910..1929 then 'New Orleans Jazz'
  when 1930..1939 then 'Swing'
  when 1940..1950 then 'Bebop'
  else 'Jazz'
  end

result =
  if some_cond
    calc_something
  else
    calc_something_else
  end
```

* 当给方法的参数赋默认值时，在 = 两边使用空格
``` ruby
# bad
def some_method(arg1=:default, arg2=nil, arg3=[])
  # do something...
end

# good
def some_method(arg1 = :default, arg2 = nil, arg3 = [])
  # do something...
end
```

* 大数值添加下划线来提高它们的可读性。
``` ruby
# bad - how many 0s are there?
num = 1000000

# good - much easier to parse for the human brain
num = 1_000_000
```

* 每一行限制在 80 个字符内。

### 语法

* 避免在不需要的地方使用 self(它仅仅在调用一些 self 做写访问的时候需要)
    (It is only required when calling a self write accessor.)
``` ruby
# bad
def ready?
  if self.last_reviewed_at > self.last_updated_at
    self.worker.update(self.content, self.options)
    self.status = :in_progress
  end
  self.status == :verified
end

# good
def ready?
  if last_reviewed_at > last_updated_at
    worker.update(content, options)
    self.status = :in_progress
  end
  status == :verified
end
```    

* 不要在条件表达式里使用 = （赋值）的返回值，除非条件表达式在圆括号内被赋值。
    这是一个相当流行的 ruby 方言，有时被称为 safe assignment in condition。
``` ruby
# bad (+ a warning)
if v = array.grep(/foo/)
  do_something(v)
  ...
end

# good (MRI would still complain, but RuboCop won't)
if (v = array.grep(/foo/))
  do_something(v)
  ...
end

# good
v = array.grep(/foo/)
if v
  do_something(v)
  ...
end
```

* 用 proc 而不是 Proc.new。

``` ruby
# bad
p = Proc.new { |n| puts n }

# good
p = proc { |n| puts n }
```    

### 类与模块

* 在 class 定义里使用一致的结构。
``` ruby
class Person
  # extend and include go first
  extend SomeModule
  include AnotherModule

  # constants are next
  SOME_CONSTANT = 20

  # afterwards we have attribute macros
  attr_reader :name

  # followed by other macros (if any)
  validates :name

  # public class methods are next in line
  def self.some_method
  end

  # followed by public instance methods
  def some_method
  end

  # protected and private methods are grouped near the end
  protected

  def some_protected_method
  end

  private

  def some_private_method
  end
end
```

* 倾向使用 module，而不是只有类方法的 class。类别应该只在创建实例是合理的时候使用。
``` ruby
# bad
class SomeClass
  def self.some_method
    # body omitted
  end

  def self.some_other_method
  end
end

# good
module SomeClass
  module_function

  def some_method
    # body omitted
  end

  def some_other_method
  end
end
```

* 当你希望将模块的实例方法变成 class 方法时，偏爱使用 module_function 胜过 extend self。

### 集合

* 用 Hash#key? 不用 Hash#has_key? 以及用 Hash#value?, 不用 Hash#has_value?  
    Matz 提到过 长的形式在考虑被弃用。
```ruby
# bad
hash.has_key?(:test)
hash.has_value?(value)

# good
hash.key?(:test)
hash.value?(value)
```    
