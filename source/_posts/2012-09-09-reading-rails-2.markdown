---
layout: post
title: "Rails 源码学习 2 (Rack and Webrick)"
date: 2012-09-09 21:33
comments: true
categories: rails
---



### 先来一个rack程序的简单例子

```ruby
require 'rubygems'
require 'rack'


class HelloWorld
  def call(env)
    return [200, {}, ["Hello world!"]]
  end
end

Rack::Handler::WEBrick.run(HelloWorld.new,:Port => 9000)
```

`HelloWorld` 方法里定义了一个call方法，call方法只有一个参数env, 方法会return一个数组，里面包含了http请求的response. 运行一下这个文件看看：

```ruby
$ ruby myserver.rb
[2012-08-24 12:24:28] INFO  WEBrick 1.3.1
[2012-08-24 12:24:28] INFO  ruby 1.9.3 (2012-04-20) [x86_64-darwin11.4.0]
[2012-08-24 12:24:28] INFO  WEBrick::HTTPServer#start: pid=17920 port=9000
```
我们通过 WebBrick的run方法启动WebBrick Server, 并且传递了HelloWorld的实例， 现在端口在9000, 打开网页访问 `http://localhost:9000` 就会看到页面里显示的是 `Hello world!`.

### rackup
rack 本身就提供了一个 rackup 工具， 我们可以用rackup工具启动一个文件，不过这个文件要是 .ru后缀结尾的，默认名称是config.ru, 这个文件是不是很眼熟, 其实在每个rails项目的根目录下都有这个文件。让我们来写一个config.ru文件

```ruby
run Proc.new {|env| [200, {"Content-Type" => "text/html"}, ["Hello Rack!"]]}
```

好了，在文件目录里运行一下

```ruby
$ rackup config.ru
[2012-08-24 14:56:34] INFO  WEBrick 1.3.1
[2012-08-24 14:56:34] INFO  ruby 1.9.3 (2012-04-20) [x86_64-darwin11.4.0]
[2012-08-24 14:56:34] INFO  WEBrick::HTTPServer#start: pid=18286 port=9292
``` 
这回是绑定在9292端口，rack的默认端口上。


### middleware

rack 的middleware 提供对任何请求的filter功能，比如我们可以为每次请求记录log, 给每个html页面里加入一些信息，有了middleware接口，
实现起来就非常方便了，而且我们可以堆叠所有的middleware,每一个middleware处理完后传给下一个，直到没有middleware了才会把结果输出出来,
把上面的例子再改一改，为每次输出里面都增加一个字符串：

```ruby
class InsertName
    def initialize(app)
      @app = app
    end

    def call(env)
      status, headers, response = @app.call(env)
      [status,headers, [response[0]+ ' jonny']]
    end 
end

use InsertName

run Proc.new {|env| [200, {"Content-Type" => "text/html"}, ["Hello Rack!"]]}
```

重新用rackup启动，访问 localhost:9292， 会看到结果是 `Hello Rack! jonny`

middleware的应用非常灵活强大，rails的很多功能都是基于rack middleware实现的，在rails项目下输入
`rack middleware` 会列出项目用到的所有middleware, 下面是我的一个项目的middleware列表：

```ruby
use ActionDispatch::Static
use Rack::Lock
use #<ActiveSupport::Cache::Strategy::LocalCache::Middleware:0x007facbb14adb0>
use Rack::Runtime
use Rack::MethodOverride
use ActionDispatch::RequestId
use Rails::Rack::Logger
use ActionDispatch::ShowExceptions
use ActionDispatch::DebugExceptions
use ActionDispatch::RemoteIp
use ActionDispatch::Reloader
use ActionDispatch::Callbacks
use ActiveRecord::ConnectionAdapters::ConnectionManagement
use ActiveRecord::QueryCache
use ActionDispatch::Cookies
use ActionDispatch::Session::CookieStore
use ActionDispatch::Flash
use ActionDispatch::ParamsParser
use ActionDispatch::Head
use Rack::ConditionalGet
use Rack::ETag
use ActionDispatch::BestStandardsSupport
use Warden::Manager
use OmniAuth::Strategies::Weibo
run Opinion::Application.routes
```

### rack handler

rack handler 是rack链接server的桥梁，rack 库自带了很多server端的handler, 我们用webrick的handler做示例，在第一个例子里我们就是用的handler启动的Webrick server：
`Rack::Handler::WEBrick.run(HelloWorld.new,:Port => 9000)`

* rack/lib/rack/handler/webrick.rb

```ruby
module Rack
  module Handler
    class WEBrick < ::WEBrick::HTTPServlet::AbstractServlet
	  def self.run(app, options={})
        options[:BindAddress] = options.delete(:Host) if options[:Host]
        @server = ::WEBrick::HTTPServer.new(options)
        @server.mount "/", Rack::Handler::WEBrick, app
        yield @server  if block_given?
        @server.start
      end

	  ...
  
  	end
  end
end
```
WEBrick类是继承于 `WEBrick::HTTPServlet::AbstractServlet` 这个类的，在 `self.run` 方法中首先是 new 了一个 WEBrick::HTTPServer，server的mount方法 加载了WEBrick handler类，接下来server处理完block里面的内容后进行启动，看来所有的操作都是 `WEBrick::HTTPServer` 类完成的，下面就到这个类的源码里看看。

Webrick是ruby的标准函数库，不在gems目录里，我的机器上在 `.rvm/rubies/ruby-1.9.3-p194/lib/ruby/1.9.1/webrick` 目录下面：

* webrick/httpserver.rb

```ruby
class HTTPServer < ::WEBrick::GenericServer
    def initialize(config={}, default=Config::HTTP)
      super(config, default)
      @http_version = HTTPVersion::convert(@config[:HTTPVersion])

      @mount_tab = MountTable.new
      if @config[:DocumentRoot]
        mount("/", HTTPServlet::FileHandler, @config[:DocumentRoot],
              @config[:DocumentRootOptions])
      end

      unless @config[:AccessLog]
        @config[:AccessLog] = [
          [ $stderr, AccessLog::COMMON_LOG_FORMAT ],
          [ $stderr, AccessLog::REFERER_LOG_FORMAT ]
        ]
      end

      @virtual_hosts = Array.new
    end

    ...
end
```

父类GenericServer主要做了一些config初始化，端口绑定的工作，mount serverlet 到 相应的路径。

然后看看 `start` 方法, httpserver里并没有start方法，而是调用了父类的start：

```ruby
def start(&block)
      raise ServerError, "already started." if @status != :Stop
      server_type = @config[:ServerType] || SimpleServer

      server_type.start{
        @logger.info \
          "#{self.class}#start: pid=#{$$} port=#{@config[:Port]}"
        call_callback(:StartCallback)

        thgroup = ThreadGroup.new
        @status = :Running
        while @status == :Running
          begin
            if svrs = IO.select(@listeners, nil, nil, 2.0)
              svrs[0].each{|svr|
                @tokens.pop          # blocks while no token is there.
                if sock = accept_client(svr)
                  sock.do_not_reverse_lookup = config[:DoNotReverseLookup]
                  th = start_thread(sock, &block)
                  th[:WEBrickThread] = true
                  thgroup.add(th)
                else
                  @tokens.push(nil)
                end
              }
            end
          rescue Errno::EBADF, IOError => ex
            # if the listening socket was closed in GenericServer#shutdown,
            # IO::select raise it.
          rescue Exception => ex
            msg = "#{ex.class}: #{ex.message}\n\t#{ex.backtrace[0]}"
            @logger.error msg
          end
        end

        @logger.info "going to shutdown ..."
        thgroup.list.each{|th| th.join if th[:WEBrickThread] }
        call_callback(:StopCallback)
        @logger.info "#{self.class}#start done."
        @status = :Stop
      }
end
```

start方法内通过@status 状态一直做循环监听，如果有请求就进入处理过程，在start_thread方法中调用httpserver 的 run方法, run方法种初始化了 `HTTPResponse` 和 `HTTPRequest` 最后传递给了service方法

```ruby
    def service(req, res)
      if req.unparsed_uri == "*"
        if req.request_method == "OPTIONS"
          do_OPTIONS(req, res)
          raise HTTPStatus::OK
        end
        raise HTTPStatus::NotFound, "`#{req.unparsed_uri}' not found."
      end

      servlet, options, script_name, path_info = search_servlet(req.path)
      raise HTTPStatus::NotFound, "`#{req.path}' not found." unless servlet
      req.script_name = script_name
      req.path_info = path_info
      si = servlet.get_instance(self, *options)
      @logger.debug(format("%s is invoked.", si.class.name))
      si.service(req, res)
    end
```

servlet 变量就是之前在webrick handler里mount的类名，在这里取回来后通过 `servlet.get_instance` 创建了 `Rack::Handler::WEBrick` 实例，
最终实例调用自己的service方法，我们又回到了 webrick handler 里面：

```ruby
def service(req, res)
  ...
  status, headers, body = @app.call(env)
  ...
end
```

在这里我们一开始传进去的app被调用到，并返回对应的内容，到此为止一个request通过rack 再到WebBrick的处理完成.




