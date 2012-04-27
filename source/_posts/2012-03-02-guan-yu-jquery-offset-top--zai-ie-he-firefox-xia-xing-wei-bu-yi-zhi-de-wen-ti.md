--- 
categories: 
  - 杂货铺
comments: true
layout: post
published: true
status: publish
tags: 
  - js
title: "关于Jquery offset().top 在IE和Firefox下行为不一致的问题"
type: post
---
项目中用到了JSTree,原先一个页面里包含了很多的tab标签，而且每个tab标签内功能都很多，用到了大量的js代码，后来我们决定把tab里的内容分成不同的page，然后用iframe调用。

于是奇怪的问题出现了，在IE下面拖动滚动条后jstree的node怎么也点不了了，像事件丢失了一样。调试了一会貌似不是我代码的原因，于是到JSTree的源码里看看，结果看到下面这一句，他计算的事件点击在页面的position要在node的li高度范围内，这个不知道为啥要这样做，很是奇怪，但是这就是为什么在iframe里会出怪问题的原因。

{% codeblock %}
if(trgt.is("ins") && event.pageY - trgt.offset().top < this.data.core.li_height)
{
    this.toggle_node(trgt);
}
{% endcodeblock %}

在ie下面当右边滚动条滚动后$(element).offset().top计算出来的值就不对了，必须要加document.documentElement.scrollTop 的值才能得到正确的结果，迫不得已，修补bug如下


{% codeblock %}

var posY = 0;
if(window.parent!= window && $.browser.msie)
    posY= (trgt.offset().top+document.documentElement.scrollTop);
else
    posY= trgt.offset().top;
if(trgt.is("ins") && event.pageY - posY < this.data.core.li_height) { this.toggle_node(trgt); }

{% endcodeblock %}

Jquery里也有人提到这个问题，新版的JSTree里已经修复此bug， 附上地址供参考
