--- 
categories: []
comments: true
layout: post
published: true
status: publish
tags: []
title: "vim window下显示换行符的问题"
type: post
---
 

关于回车与换行

很久以前，老式的电传打字机使用两个字符来另起新行。一个字符把滑动架移回首位 (称为回车，<CR>，ASCII码为0D)，另一个字符把纸上移一行 (称为换行, <LF>，ASCII码为0A)。当计算机问世以后，存储器曾经非常昂贵。有些人就认定没必要用两个字符来表示行尾。UNIX 开发者决定他们可以用 一个字符来表示行尾，Linux沿袭Unix，也是<LF>。Apple 开发者规定了用<CR>。开发 MS-DOS以及Windows 的那些家伙则决定沿用老式的<CR><LF>。

因为MS-DOS及Windows是回车＋换行来表示换行，因此在Linux下用Vim查看在Windows下用VC写的代码，行尾后的“^M”符号，表示的是符。

在Vim中解决这个问题，很简单，在Vim中利用替换功能就可以将“^M”都干掉，键入如下替换命令行：
<div>:%s/^M//g</div>
注意：上述命令行中的“^M”符，不是“^”再加上“M”，而是由“Ctrl+v”、“Ctrl+M”键生成的

 

vim 自动识别换行符

set fileformats=unix,dox,mac

设置保存格式

set fileformat=unix

 
