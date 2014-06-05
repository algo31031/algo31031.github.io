---
layout: post
title: "The RSpec Book笔记(七) RSpec的工具、集成与扩展"
date: 2014-06-05 14:29:11 +0800
comments: true
categories: [test, RSpec]
---
### The rspec Command
``` bash
rspec simple_math_spec.rb  #执行单个文件
rspec spec                 #执行spec文件夹下全部文件 
```

** --format,设置输出格式 **
``` bash
rspec path/to/my/specs --format documentation
rspec path/to/my/specs --format html:path/to/my/report.html
rspec path/to/my/specs  --format progress \
                        --format nested:path/to/my/report.txt \
                        --format html:path/to/my/report.html
```

** 其他 **
``` bash
rspec spec --backtrace
rspec spec --color
```

### Rake
``` bash
rake spec               #执行spec下的全部specs文件
rake spec:controllers   #执行spec下的全部specs/controllers文件
```

完整命令列表可以通过执行`rake -T spec`获取, 这些命令被定义在`RSpec::Core::RakeTask`

可以在`Rakefile`里添加以下内容作来配置以rake执行的rspec
``` ruby
require 'rspec/core/rake_task'

RSpec::Core::RakeTask.new do |t|
  t.rspec_opts = ["--color", "--format", "specdoc"]
end
```

### Filtering

** Inclusion **
``` ruby
RSpec.configure do |c|
  c.filter = { :focus => true }
end

describe "group" do
  it "example 1", :focus => true do
  end

  it "example 2" do
  end
end
```
上面这段代码执行后会得到类似下面的输出
``` bash
group
  example 2

Finished in 0.00067 seconds
1 example, 0 failures
```

其中`it()`里的`:focus => true`部分被称为metadata  
值可以通过`example.metadata[:focus]`获取到

** Exclusion **
``` ruby
RSpec.configure do |c|
c.exclusion_filter = { :slow => true }
end

describe "group" do
  it "example 1", :slow => true do
  end

  it "example 2" do
  end
end
```
执行时"example 2"被排除在外

** Lambdas **

Inclusion与Exclusion的filter都可以接受一个lambda处理更复杂的逻辑

``` ruby
require 'ping'

RSpec.configure do |c|
  c.exclusion_filter = {
    :if => lambda {|what|
      case what
      when :network_available
        !Ping.pingecho "example.com", 10, 80
      end
    }
  }
end

describe "network group" do
  it "example 1", :if => :network_available do
  end

  it "example 2" do
  end
end
```
