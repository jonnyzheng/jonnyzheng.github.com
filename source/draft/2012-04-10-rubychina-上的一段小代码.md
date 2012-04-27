--- 
categories: []
comments: true
layout: post
published: false
status: draft
tags: []
title: "rubychina 上的一段小代码"
type: post
---
如下的代码会输出什么内容


{% codeblock %}

module Foo
  def self.included(base)
    base.class_eval do
      extend ClassMethods
      include InstanceMethods
    end
  end

  module ClassMethods
    def hello
      new.world
    end
  end

  module InstanceMethods
    def world
      puts "Hello, world"
    end
  end
 end

class Bar
  include Foo
  hello
end


{% endcodeblock %}
