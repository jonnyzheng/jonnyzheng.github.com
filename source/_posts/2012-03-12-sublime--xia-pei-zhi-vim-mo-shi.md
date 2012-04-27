--- 
categories: 
  - 杂货铺
comments: true
layout: post
published: true
status: publish
tags: 
  - 编辑器
title: "Sublime 下配置vim模式"
type: post
---
最近用上了sublime text2， 和textmate比界面要漂亮一些，而且几个平台下都有对应版本，比较统一。

sublime支持文本编辑使用 vim 模式，vim 快捷键编辑文本还是挺快的，两个编辑器融合一下也挺好，选择Preferences->Settings- Default, 在文本的最下面有一行 
{% codeblock %}
"ignored_packages": ["vintage"]
{% endcodeblock %}
 ，这里sublime 默认去掉了vim的支持，我们只需要把"vintage"删掉就好了。

再在编辑框里试试已经ok了，但是在vim里我都把ESC键映射到了'ii'上了： imap 'ii' <esc> ，这里并不支持imap。 不过我们可以在vintage 的package包里自己定义，我的机器是windows,默认packages都装在了 C:\Documents and Settings\zhengj1\Application Data\Sublime Text 2\Packages 目录下，找到Vintage\Default.sublime-keymap文件，用文本编辑器打开，加上下面的代码：


{% codeblock %}
"ignored_packages": ["vintage"]
{% endcodeblock %}


这个keymap文件里可以定义自己习惯的快捷键方式，有兴趣的不妨研究研究。

vintage这个插件并不支持command， 想要支持还需要下一个<a href="https://github.com/SublimeText/VintageEx" title="VintageEx">VintageEx </a>包，一些简单的命令就可以用了。</esc>
