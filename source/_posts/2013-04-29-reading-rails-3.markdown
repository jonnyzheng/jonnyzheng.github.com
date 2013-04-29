---
layout: post
title: "Rails 源码学习3 (Application,Engine,Railtie)"
date: 2013-04-29 13:05
comments: true
categories: 
---


在第一篇[Rails 的初始化过程](http://jonnyzheng.github.com/blog/2012/07/15/reading-rails-1/)里讲到rails的启动命令是在 `railties/lib/rails/commands/server.rb` 里执行的，其实这里的server类是继承自`Rack::Server`的，从这里就可以看出rails是一个rack app，rack 连接了Rails应用和 WebServer, 现在让我们再深入进去看看rails的各个组件是如何被整合在一起的。

在 Rails::Server类里一些方法被覆写，用于加载rails自己的config环境，middleware等：

```ruby
# 覆盖了rack的middleware 方法，加载了3个middleware app
def middleware
      middlewares = []
      middlewares << [Rails::Rack::LogTailer, log_path] unless options[:daemonize]
      middlewares << [Rails::Rack::Debugger]  if options[:debugger]
      middlewares << [::Rack::ContentLength]
      Hash.new(middlewares)
end




# 默认端口改成3000，定义了rack的启动文件config.ru, 还有一些默认配置
def default_options
      super.merge({
        :Port        => 3000,
        :environment => (ENV['RAILS_ENV'] || "development").dup,
        :daemonize   => false,
        :debugger    => false,
        :pid         => File.expand_path("tmp/pids/server.pid"),
        :config      => File.expand_path("config.ru")
      })
end
```


## rails项目里的 config.ru 
rack最后会执行config.ru,然后看看rails项目的根目录下config.ru里是如何写的

```ruby
#引用了config下面的 enviroment.rb 
require ::File.expand_path('../config/environment',  __FILE__)

# RailsStudy是我的rails的项目名称，这里的工作就是run 这个Application
run RailsStudy::Application
```
我们的Rails代码就是在这一步开始被调用到的  



下面来看看 `config/application.rb` 这个文件

```ruby
require File.expand_path('../boot', __FILE__)

require 'rails/all'

if defined?(Bundler)
  # If you precompile assets before deploying to production, use this line
  Bundler.require(*Rails.groups(:assets => %w(development test)))
  # If you want your assets lazily compiled in production, use this line
  # Bundler.require(:default, :assets, Rails.env)
end

module RailsStudy
  class Application < Rails::Application
    # Settings in config/environments/* take precedence over those specified here.
    # Application configuration should go into files in config/initializers
    # -- all .rb files in that directory are automatically loaded.

    # Custom directories with classes and modules you want to be autoloadable.
    # config.autoload_paths += %W(#{config.root}/extras)

    # Only load the plugins named here, in the order given (default is alphabetical).
    # :all can be used as a placeholder for all plugins not explicitly named.
    # config.plugins = [ :exception_notification, :ssl_requirement, :all ]

    # Activate observers that should always be running.
    # config.active_record.observers = :cacher, :garbage_collector, :forum_observer

    # Set Time.zone default to the specified zone and make Active Record auto-convert to this zone.
    # Run "rake -D time" for a list of tasks for finding time zone names. Default is UTC.
    # config.time_zone = 'Central Time (US & Canada)'

    # The default locale is :en and all translations from config/locales/*.rb,yml are auto loaded.
    # config.i18n.load_path += Dir[Rails.root.join('my', 'locales', '*.{rb,yml}').to_s]
    config.i18n.default_locale = "zh-CN"

    # Configure the default encoding used in templates for Ruby 1.9.
    config.encoding = "utf-8"

    # Configure sensitive parameters which will be filtered from the log file.
    config.filter_parameters += [:password]

    # Enable escaping HTML in JSON.
    config.active_support.escape_html_entities_in_json = true

    # Use SQL instead of Active Record's schema dumper when creating the database.
    # This is necessary if your schema can't be completely dumped by the schema dumper,
    # like if you have constraints or database-specific column types
    # config.active_record.schema_format = :sql

    # Enforce whitelist mode for mass assignment.
    # This will create an empty whitelist of attributes available for mass-assignment for all models
    # in your app. As such, your models will need to explicitly whitelist or blacklist accessible
    # parameters by using an attr_accessible or attr_protected declaration.
    config.active_record.whitelist_attributes = true

    # Enable the asset pipeline
    config.assets.enabled = true

    # Version of your assets, change this if you want to expire all your assets
    config.assets.version = '1.0'
  end
end
```
这是一个默认的application文件，基本没有修改里面的内容. 在rails项目里application文件的作用主要是做一些初始化的设置，比如时区，i18n等等。

`RailsStudy::Application` 类是继承自 `Rails::Application`,然后我们去看看这个文件在哪里

### railties/lib/rails/application.rb

application文件里做了详细的介绍 Application的功能：

* Initialization:  
   Rails::Application负责执行所有的railties,engines,plugin的 initializer, 并且也执行
   bootstrap initializer 和 finishing initializer

* Configureation:  
  负责设置App的config，其中包含了 Rails::Engine和Rails::Railtie的config
  
* Routes:  
  负责管理routes,当文件改变的时候reload所有的routes
   
* Middlewares:  
   负责构建middlewares的堆栈

* Booting process:  
  负责按照启动顺序启动和执行程序，大致程序如下
  1. require "config/boot.rb" to setup load paths
  2. require railties and engines
  3. Define Rails.application as "class MyApp::Application < Rails::Application"
  4. Run config.before_configuration callbacks
  5. Load config/environments/ENV.rb
  6. Run config.before_initialize callbacks
  7. Run Railtie#initializer defined by railties, engines and application.
  8. One by one, each engine sets up its load paths, routes and runs its config/initializers/* files.
  9. Custom Railtie#initializers added by railties, engines and applications are executed
  10. Build the middleware stack and run to_prepare callbacks
  11. Run config.before_eager_load and eager_load if cache classes is true
  12. Run config.after_initialize callbacks
  
  
  application里面的call 方法用来给rack做调用的
  
  
### railties/lib/rails/engine.rb
Rails::Engine用来生成一个简单的Rails应用或子集，打包成通用的程序供不同的Rails项目使用。
从Rails 3.0开始Rails应用就是一个Engine, 实际上Application类是继承自Engine类的。


  
  
### railties/lib/rails/railtie.rb

Rails::Application < Rails::Engine < Rails::Railtie  

Railties 是rails framework的核心，用来提供各种回调方法以及扩展，或者修改初始化的process. 实际上每个rails组件(ActionMailer,Action Controller)都是一个railtie, 每个组件都有自己的初始化代码，这样rails就不用去专门把初始化工作都自己来做，只要在正确的时候调用各组件的初始化代码就可以了。


Railtie类中一开始就引用了几个类：

```ruby
autoload :Configurable,  "rails/railtie/configurable"
autoload :Configuration, "rails/railtie/configuration"

include Initializable
```

#### Configurable & Configuration
rails/railtie/configurable 中的类方法moudule中定义了 instance方法，并且将 @instance的config 方法代理给了Railtie类，

```ruby
module Rails
  class Railtie
    module Configurable
      extend ActiveSupport::Concern

      module ClassMethods
          
        delegate :config, :to => :instance

        def inherited(base)
          raise "You cannot inherit from a #{self.superclass.name} child"
        end

        def instance
          @instance ||= new
        end

        ...
      end
    end
  end
end
```

这里的@instance有可能是Railtie，Engine,Application类的实例，他们都有自己的config方法实现,方法里基本上都是生成了自己的
Configuration实现，这些Configuration类也是依次的继承关系:
`Application::Configuration < Engine::Configuration < Railtie::Configuration`


#### Initializable
Initializable类用来让任何include他的类都可以使用initializer方法，在Engine类中我们可以看到很多调用initializer的code,
比如用来初始化i18n的本地化文件路径:

```ruby
initializer :add_locales do
  config.i18n.railties_load_path.concat(paths["config/locales"].existent)
end
```
所有的initializer会被加入到这个类的initializers变量中，最后在Application类中的initialize!方法中会被执行



