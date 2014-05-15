---
layout: post
title: "Ruby下的 ===, ==, eql?, equal?"
date: 2014-05-14 16:18:04 +0800
comments: true
categories: ruby
---
### === case equality

严格说来, 这个其实跟另外三个不属于一类.  

` a === b` 相当于 "如果我有一个贴了a标签的抽屉, 那么把b放到这个抽屉里是否可行?"

作为case equality operator, 其首要作用自然是在`when/case`里, 作为判断进入某个分支的依据

``` ruby
a = 5

case a
when Integer
  "case Integer"
when 5
  "case 5"
end
```

这段代码其实是会返回`"case Integer"`的, 因为`Integer`类重写了`===`方法
``` irb
2.1.1 :043 > Integer === 5
 => true 
```

相应的, 重写了`===`的还有`Range`, `Regexp`等等
``` irb
2.1.1 :054 > (1..5) === 5
 => true 
2.1.1 :055 > (1...5) === 5
 => false 
2.1.1 :058 > /(a|e|i|o|u)/ === "hello"
 => true  
```

对于`Object#===`, 事实上等价于`Object#==`.  
但是对Object的子类, `===`一般会被重写, 使其在条件表达式里有意义.

这里可以拿`<=>`方法做个类比

比如我们自己定义了个`User`类, 如果直接就`User.all.sort`, 是不可以的,  
如果想这么用, 你就需要首先在`User`里面定义一个`<=>`方法用于比较2个user对象

稍微有点不同的是, 因为`===`是定义在`Object`里的  
所以即便对自己定义的类里面不重写`===`方法, 一样可以对其使用`case/when`  
但是这样会调用Object#===, 可能不会得到自定义类需要的结果.

依然用上面的例子来说
如果`Integer`里面没有重写`===`方法, 那么得到的返回值就不再是"case Integer",而是"case 5"了

### ==

`==`是最为常用的用于比较两个对象值是否相等的方法

`a == b`相当于判断"a的值与b的值相同吗"

`==`方法经常被Object子类重写以满足其自身需求

### equal?

`equal?`方法用来判断2个对象是否是同一个对象

`a.equal? b`相当于判断"a就是b吗"

``` irb
2.1.1 :005 > "a" == "a"
 => true 
2.1.1 :006 > "a".equal? "a"
 => false 
2.1.1 :007 > :a.equal? :a
 => true 
```

与`==`不同, `equal?`不应该被子类重写

### eql? 

与`==`类似, 但是可以看做是更严格的`==`  
在`Object`里, `==`与`eql?`是同意, 但是很多子类会重写`eql?`以提供更严格的比较, 比如:
``` irb
2.1.1 :010 > 1 == 1
 => true 
2.1.1 :011 > 1 == 1.0
 => true 
2.1.1 :012 > 1.eql? 1.0
 => false 
```



