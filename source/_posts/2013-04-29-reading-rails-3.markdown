---
layout: post
title: "Rails 源码学习3 (Application,Engine,Railtie)"
date: 2013-04-29 13:05
comments: true
categories: rails
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

## railties/lib/rails/application.rb

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
    
    
 这里先介绍一下application的功能，到最后我们再去看代码，接下来先看看Railtie。  
  

  
  
## railties/lib/rails/railtie.rb

Rails::Application < Rails::Engine < Rails::Railtie  

Railties 是rails framework的核心，用来提供各种回调方法以及扩展，或者修改初始化的process. 实际上每个rails组件(ActionMailer,Action Controller)都是一个railtie, 每个组件都有自己的初始化代码，这样rails就不用去专门把初始化工作都自己来做，只要在正确的时候调用各组件的初始化代码就可以了。




Railtie的代码其实很短，只有100多行，类中一开始就引用了下面几个模块：

```ruby
autoload :Configurable,  "rails/railtie/configurable"
autoload :Configuration, "rails/railtie/configuration"

include Initializable
```

#### Configurable & Configuration
rails/railtie/configurable 中的类方法moudule中定义了 instance方法，并且将 @instance的config 方法代理给了include自己的类，

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

这里的@instance有可能是Railtie，Engine,Application类的实例，他们都有自己的config方法实现,方法里基本上都是生成了自己的Configuration实现，这些Configuration类也是依次的继承关系:
`Application::Configuration < Engine::Configuration < Railtie::Configuration`


#### Initializable

railties/lib/rails/rails/initializable.rb 中定义了Module Rails::Initializable, 所有include了该Module的类都拥有了 `initializer`,`initializers`,`initializers_chain` 等类方法，`initializer` 方法帮助类可以定义一个或多个自己的 initializer, 并且通过`initializers`,`initializers_chain`来获取他们。


比如用来加载i18n的本地化文件路径:

```ruby
initializer :add_locales do
  config.i18n.railties_load_path.concat(paths["config/locales"].existent)
end
```


```ruby
def initializer(name, opts = {}, &blk)
        raise ArgumentError, "A block must be passed when defining an initializer" unless blk
        opts[:after] ||= initializers.last.name unless initializers.empty? || initializers.find { |i| i.name == opts[:before] }
        initializers << Initializer.new(name, nil, opts, &blk)
end
```

在方法中会生成一个`Initializer`的实例，然后加入到`@initializers`变量中，最后会在Application类中的initialize!方法中会被执行。


## railties/lib/rails/engine.rb

Rails::Engine用来生成一个简单的Rails应用的子集，打包成通用的程序供不同的Rails项目使用。
从Rails 3.0开始Rails应用就是一个Engine, 实际上Application类是继承自Engine类的。

我们写的Gem包也可以是一个Engine应用，里面可以包含自己的controller,model,view文件，文件结构也可以和rails的文件结构一样，这样的构架很方便做一些通用的功能包，为减少重复代码提供了极大的方便。Engine本身还是有很多内容可以说的，关于如何应用他可以看看这个教程：[Getting Started with Engines](http://guides.rubyonrails.org/engines.html)。

再回过头来看代码，首先 Engine类是继承于 Railtie 的，也就是拥有了Railtie类的基本功能，但同时Engine类也扩展了自己的configuration类，这里主要为一个Engine定义了他的文件结构：


railties/lib/rails/engine/configuration.rb

```ruby
      def paths
        @paths ||= begin
          paths = Rails::Paths::Root.new(@root)
          paths.add "app",                 :eager_load => true, :glob => "*"
          paths.add "app/assets",          :glob => "*"
          paths.add "app/controllers",     :eager_load => true
          paths.add "app/helpers",         :eager_load => true
          paths.add "app/models",          :eager_load => true
          paths.add "app/mailers",         :eager_load => true
          paths.add "app/views"
          paths.add "lib",                 :load_path => true
          paths.add "lib/assets",          :glob => "*"
          paths.add "lib/tasks",           :glob => "**/*.rake"
          paths.add "config"
          paths.add "config/environments", :glob => "#{Rails.env}.rb"
          paths.add "config/initializers", :glob => "**/*.rb"
          paths.add "config/locales",      :glob => "*.{rb,yml}"
          paths.add "config/routes",       :with => "config/routes.rb"
          paths.add "db"
          paths.add "db/migrate"
          paths.add "db/seeds",            :with => "db/seeds.rb"
          paths.add "vendor",              :load_path => true
          paths.add "vendor/assets",       :glob => "*"
          paths.add "vendor/plugins"
          paths
        end
      end
```




railties/lib/rails/engine/railties.rb

Engine的Railties类是用来获取相关的railties，如self.railies方法可以获得当前环境下所有
继承了`::Rails::Railtie` 的类的实例, self.engines则可以得到所有继承`::Rails::Engine`
的实例。




回到engine.rb, 这里定义了很多initializer,大部分的initializer是用来查看并加载所需要的文件用的。

```
set_autoload_paths #设置需要自动load的文件路径
add_routing_paths #如果routes文件存在则load相关文件
add_locales  #如果本地化文件存在则load相关文件
load_config_initializers #如果有initializer文件存在则load相关文件
```




## railties/lib/rails/application.rb

了解完Engine后再来看看Application的代码, 首先是


```ruby
class Application < Engine
    autoload :Bootstrap,      'rails/application/bootstrap'
    autoload :Configuration,  'rails/application/configuration'
    autoload :Finisher,       'rails/application/finisher'
    autoload :Railties,       'rails/application/railties'
    autoload :RoutesReloader, 'rails/application/routes_reloader'


	...
end    
```

rails/application/bootstrap 定义启动是最开始需要加载的initializers,比如加载Logger,加载ActiveSupport的所有功能。

rails/application/configuration 是继承于 ::Rails::Engine::Configuration，这里他提供了一个Rails Application所需要的更多config options:

```ruby
module Rails
  class Application
    class Configuration < ::Rails::Engine::Configuration
      attr_accessor :allow_concurrency, :asset_host, :asset_path, :assets,
                    :cache_classes, :cache_store, :consider_all_requests_local,
                    :dependency_loading, :exceptions_app, :file_watcher, :filter_parameters,
                    :force_ssl, :helpers_paths, :logger, :log_tags, :preload_frameworks,
                    :railties_order, :relative_url_root, :reload_plugins, :secret_token,
                    :serve_static_assets, :ssl_options, :static_cache_control, :session_options,
                    :time_zone, :reload_classes_only_on_change, :whiny_nils


   ...
end
```


更多的paths:

```ruby
 def paths
        @paths ||= begin
          paths = super
          paths.add "config/database",    :with => "config/database.yml"
          paths.add "config/environment", :with => "config/environment.rb"
          paths.add "lib/templates"
          paths.add "log",                :with => "log/#{Rails.env}.log"
          paths.add "public"
          paths.add "public/javascripts"
          paths.add "public/stylesheets"
          paths.add "tmp"
          paths
        end
      end

```

rails/application/finisher.rb 用来定义在启动完成前需要加载的initializers
rails/application/railties.rb 用来获得所有的railties，包括engines,plugins,railties。
rails/application/routes_reloader.rb 用来提供检测文件改动和reload routes的功能。


initialize!方法在每个Rails项目的config/enviroment.rb中被调用到，这时会真正的执行所有定义好
的initializers

```ruby
def initialize!(group=:default) #:nodoc:
    raise "Application has been already initialized." if @initialized
    run_initializers(group, self)
    @initialized = true
    self
end
```

每一个initializer最后在 Rails::Initializable::Initializer类的run方法中被执行到：

```ruby
def run(*args)
    @context.instance_exec(*args, &block)
end
```


default_middleware_stack方法为rails程序加载默认的middleware, 该方法在Engine类的`app`方法中被调用到。

```ruby

def default_middleware_stack
      ActionDispatch::MiddlewareStack.new.tap do |middleware|
        if rack_cache = config.action_controller.perform_caching && config.action_dispatch.rack_cache
          require "action_dispatch/http/rack_cache"
          middleware.use ::Rack::Cache, rack_cache
        end

        if config.force_ssl
          require "rack/ssl"
          middleware.use ::Rack::SSL, config.ssl_options
        end

        if config.serve_static_assets
          middleware.use ::ActionDispatch::Static, paths["public"].first, config.static_cache_control
        end

        middleware.use ::Rack::Lock unless config.allow_concurrency
        middleware.use ::Rack::Runtime
        middleware.use ::Rack::MethodOverride
        middleware.use ::ActionDispatch::RequestId
        middleware.use ::Rails::Rack::Logger, config.log_tags # must come after Rack::MethodOverride to properly log overridden methods
        middleware.use ::ActionDispatch::ShowExceptions, config.exceptions_app || ActionDispatch::PublicExceptions.new(Rails.public_path)
        middleware.use ::ActionDispatch::DebugExceptions
        middleware.use ::ActionDispatch::RemoteIp, config.action_dispatch.ip_spoofing_check, config.action_dispatch.trusted_proxies

        if config.action_dispatch.x_sendfile_header.present?
          middleware.use ::Rack::Sendfile, config.action_dispatch.x_sendfile_header
        end

        unless config.cache_classes
          app = self
          middleware.use ::ActionDispatch::Reloader, lambda { app.reload_dependencies? }
        end

        middleware.use ::ActionDispatch::Callbacks
        middleware.use ::ActionDispatch::Cookies

        if config.session_store
          if config.force_ssl && !config.session_options.key?(:secure)
            config.session_options[:secure] = true
          end
          middleware.use config.session_store, config.session_options
          middleware.use ::ActionDispatch::Flash
        end

        middleware.use ::ActionDispatch::ParamsParser
        middleware.use ::ActionDispatch::Head
        middleware.use ::Rack::ConditionalGet
        middleware.use ::Rack::ETag, "no-cache"

        if config.action_dispatch.best_standards_support
          middleware.use ::ActionDispatch::BestStandardsSupport, config.action_dispatch.best_standards_support
        end
      end
    end

```


嗯，基本上就是这回所有的内容了，总结一下： Railtie为所有rails组件提供了和Rails整合的接口，使他们可以定义自己的初始化内容，可以有自己的配置。 Engine继承于Railtie, 提供了一个功能最小化的Rails文档结构，Application继承于Engine, 在Engine的基础上定义了一个完整Rails所需要的Initializers和middleware,同时在启动时负责加载Rails程序中其他组件的所有Initializers。

