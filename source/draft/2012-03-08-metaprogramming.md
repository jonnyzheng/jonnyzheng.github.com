--- 
categories: []
comments: true
layout: post
published: false
status: draft
tags: []
title: metaprogramming
type: post
---




Proc和lambda的区别

Proc 的 return 是和lambda的 return 是不一样滴，lambda <strong>return</strong> 是退出自己作用域范围。

proc 的 return 是退出proc定义时所在的作用域，这个需要注意，并不是退出执行proc的method或类似作用域，而是proc定义时所在的作用域，所以看下面的例子：

proc在mytest方法里被执行，但是return实际上是作用在myclient方法上面，所以下面的123永远不会被打印出来
<script src="https://gist.github.com/1998921.js?file=define_proc_in_method.rb"></script>

这里 proc的return 是发生在最外层的，而在最外层做return操作就报错了
<script src="https://gist.github.com/1998921.js?file=define_proc_outside.rb"></script>


lambda的返回只是返回自己的作用域，比较好理解,下面的代码会继续执行mytest后面的方法，并得到结果
<script src="https://gist.github.com/1998921.js?file=lambda_sample.rb"></script>
