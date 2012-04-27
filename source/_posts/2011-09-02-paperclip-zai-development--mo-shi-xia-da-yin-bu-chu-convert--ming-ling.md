--- 
categories: 
  - "Ruby &amp; Rails"
comments: true
layout: post
published: true
status: publish
tags: 
  - paperclip
title: "Paperclip在development 模式下打印不出convert 命令"
type: post
---
今天升级了一下bundle，结果在调试时发现原先可以打印paperclip执行的imagemagick命令，现在却不可以了，网上看了一下，原因是paperclip 调用了 Cocaine插件， 而之前执行命令的log代码已经删掉了，全靠Cocaine输出。

简单的改法:

修改 config/environments/development.rb文件，最后一行加一句:


{% codeblock %}
Cocaine::CommandLine.logger = Logger.new(STDOUT)
{% endcodeblock %}
 
