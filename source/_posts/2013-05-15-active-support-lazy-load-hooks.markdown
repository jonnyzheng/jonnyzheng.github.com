---
layout: post
title: "Rails源码学习(active_support:lazy_load_hooks)"
date: 2013-05-15 23:42
comments: true
categories: rails
---


在rails的initialize代码中经常能看到方法 `ActiveSupport.on_load`，这个代码是起什么用的引起了我很大的好奇，于是看了看源码，觉得这个功能还是挺好玩而且有用的。

`ActiveSupport.on_load`方法为rails提供了一种叫做 lazy_load_hooks的功能，这个功能可以使很多组件在被用到时才被加载，而不是启动即全部加载，这样大大加快了rails app 的启动速度。

lazy_load_hooks的代码很短，只有不到30行，文件名: `activesupport-3.2.x/lib/active_suport/lazy_load_hooks.rb`

```ruby
module ActiveSupport
  @load_hooks = Hash.new { |h,k| h[k] = [] }
  @loaded = Hash.new { |h,k| h[k] = [] }

  def self.on_load(name, options = {}, &block)
    @loaded[name].each do |base|
      execute_hook(base, options, block)
    end
    @load_hooks[name] << [block, options]
  end

  def self.execute_hook(base, options, block)
    if options[:yield]
      block.call(base)
    else
      base.instance_eval(&block)
    end
  end

  def self.run_load_hooks(name, base = Object)
    @loaded[name] << base
    @load_hooks[name].each do |hook, options|
      execute_hook(base, options, hook)
    end
  end
end

```

代码中会初始化两个Hash变量 `@load_hooks`和 `@loaded`, `@load_hooks`用来存放还没有被执行的blocks, @loaded用来存放已经被加载的载体类。

让我们看看一个ActiveRecord的例子来了解一下lazy_load_hooks的使用方法，在`activerecord-3.2.x/lib/active_record/railtie.rb`文件中有很多的initializer块，这些内容在rails启动的时候都会被调用到，像下面这个加载logger的：

```ruby
initializer "active_record.logger" do
    ActiveSupport.on_load(:active_record) { self.logger ||= ::Rails.logger }
end
```

这时候 `ActiveSupport.run_load_hooks`还没有执行，所以 on_load方法中`@loaded[:active_record]`是空的，里面的each循环不会执行，执行到的是

```ruby
@load_hooks[name] << [block, options]
```
把block放进`@load_hooks[:active_record]`里面去，所以在railtie里的所有on_load的block都先被存放了起来。


最后的执行是在`activerecord-3.2.x/lib/active_record/base.rb`文件中，代码的最下面执行到了`run_load_hooks`方法，

```ruby
ActiveSupport.run_load_hooks(:active_record, ActiveRecord::Base)
```
当加载ActiveRecord::Base执行到该方法，并且把`ActiveRecord::Base`作为载体类传了进去。所以在ActiveRecord::Base被加载的时候所有的lazy_load block才会被执行到，同样的其他模块中的railite里也有很多lazy_load block，他们都是在需要的时候才被执行，这样又节省运行资源，又节省启动时间，我们自己写Gem的时候也可以用上啦。


