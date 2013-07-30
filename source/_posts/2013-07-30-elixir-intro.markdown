---
layout: post
title: "Elixir语言特性简介"
date: 2013-07-30 23:06
comments: true
categories: 
---


Erlang真是一门让人又爱又恨的语言，爱的是他天生的分布式能力，恨的是有点变态的语法，可以把人搞晕的标点符号，稍不注意就一堆大小括号的方法调用. 不过最近有一个开源项目让我小兴奋了一下，名字叫[elixir](http://elixir-lang.org). elixir是一个基于Erlang VM的编程语言，它在语法上大量借鉴了ruby的元编程思想，同时有一些自己的很有亮点的地方，而本身又与Erlang完全兼容，很好的将Erlang的语法缺点去除，保留了分布式的特性，是一门很有想象空间的语言。

接下来通过比较语法点的方式来吐槽一下Erlang语法，展示一下Elixir的优秀之处。


##atom和变量

######Erlang
Erlang中的Atom是以小写字母开始的一个单词，变量是首字母大写的单词，因为只是首字母大小写的关系很容易让人区分不清，尤其是在Tuples这样的数据结构中和变量的区分就更不明显了：

```
{a,B,C} =  Person
```
人的大脑是很有趣的，他们能够模糊的处理一些事物，通常情况下可以忽略字母的大小写认为上面的a B C性质上是一样的东西，可谁知道Erlang的定义中小写和大写有这么明显的不同，a是atom,而B C是变量。

######Elixir
Elixir用冒号加单词的方式来表示atom,变量也不需要首字母大写，都是小写的。

```
{:a,b,c} = person
```
这里的`:a`就是一个atom，是不是让你想到了ruby里的Symbol，而b,c是变量。这样的表示要清晰明确的多，妈妈再也不用担心我给搞浑了。



##混乱的结束标识

######Erlang
在Erlang中一行的结束用`.`表示，刚知道这个语法的时候感觉非常不适应，当碰到浮点数的写起来也很怪异 

```
Pi = 3.14159.
```
如果只有`.`做为结束标识我也就懒得吐槽了，问题是根据不同的控制结构还有各种各样的方式标识结束，其中包括了 `,` `;` `end` 以及无符号，虽说熟了之后也知道都是代表什么意思，但写代码的时候却总是不放心想去查查看结束符有没有用对，嗨~~~~


```
odds_and_evens(L) ->
    Odds = [X || X <- L, (X rem 2) =:= 1],
    Evens = [X || X <- L, (X rem 2) =:= 0],
    {Odds, Evens}.
```
上面这个比较简单在一个方法体内，最后一行用`.`表示结束，之前的行需要用`,`, 接下来再看个难的

```
filter(P, [H|T]) ->
	case P(H) of
		true -> [H|filter(P, T)];
		false -> filter(P, T)
	end;
filter(P, []) ->
	[].
```
是不是看着有点糊涂了，怎么有的结束是`;`, 有的没有符号,有的是 `end;`, 如果没有学过Erlang的同学可以去看看 case语法以及方法的匹配模式。总之在一个结束标识上搞出这么多花样，真是苦了我们这些码农。

######Elixir

Elixir处理结尾很简单，就是没有符号，和ruby一样，所有方法都用end结束, Less is More。

```
def hello
	IO.puts "hello world"
end

```




##比较运算符

######Erlang

```
X > Y X is greater than Y.
X < Y X is less than Y.
X =< Y X is equal to or less than Y.
X >= Y X is greater than or equal to Y.
X == Y X is equal to Y.
X /= Y X is not equal to Y.
X =:= Y X is identical to Y.
X =/= Y X is not identical to Y
```
前面几个还挺正常的，后三个就又让人有些不淡定了，不等于竟然这么写`/=`, 严格的等于是 `=:=`, 严格的不等于是`=/=`, 这种新颖的表示方式让我们这些用过不少其他语言的同学感到很不适应。


######Elixir
Elixir的处理只能用很正常来表示，呵呵：

```
a === b # strict equality (so 1 === 1.0 is false)
a !== b # strict inequality (so 1 !== 1.0 is true)
a == b # value equality (so 1 == 1.0 is true)
a != b # value inequality (so 1 != 1.0 is false)
a > b # normal comparison
a >= b # :
a < b # :
a <= b # :
```

这个点其实也没有什么太大的问题，Erlang只是没有按照大部分语言的习惯做法来罢了，如果有人觉的这样没什么不习惯那也能够理解，我其实仅仅是想知道Erlang的这种表示方式的来源或者目的是什么。


##发送消息

######Erlang

Erlang中进程间的通信是依靠互相发送消息进行的:

```
Pid ! Message
```
语法很简单，Pid是进程，Message是消息，中间用 `!` 分隔

######Elixir
Elixir对消息发送的语法只做了简单的改变，但看上去要相当人性化

```
Process <- Message
```
很显然有了`<-`在中间表达了一种指向性的涵义，根据这样的语法来理解消息的传递要容易的多。


##嵌套方法的调用

######Erlang

当在调用方法时经常会用嵌套的方法减少变量的定义,但是用多了会很难看，当然这不光是Erlang语言有这个问题，其他的语言也一样有。

```
list_to_atom(binary_to_list(capitalize_binary(list_to_binary(atom_to_list(X))))).
```

######Elixir
Elixir为了简化大量的中间变量，使用了一个叫做`Pipe Operator`的语法。看看上面那句在Elixir中我们可以这样写:

```
X |> atom_to_list |> list_to_binary |> capitalize_binary |> binary_to_list |> binary_to_atom
```
这就相当于Unix里的管道，把前一个命令的结果当做参数，传递给下个命令。在Elixir里面结果做为下一个方法的第一参数传入，产生结果后在传给下一个，直到最后。管道的使用大大优化了程序的可读性，我认为算是Elixir里面最有亮点的一个特性啦。



##字符串拼接

######Erlang
Erlang的字符串拼接是很麻烦的事情，尤其是想在一句话中间拼接变量的时候

```
Name = "jonny".
Sentence = "This is " ++ Name ++ "speaking".

```

######Elixir
这里借鉴了Ruby的表达方式，可以在双引号的字符串中连接变量

```
name = "jonny"
sentence = "This is #{name} speaking"

```




##期望
现在想想Ruby为什么会成功，就是因为他优雅的设计吸引了很多优秀的Programmer, 让更多的人真的感受到了编码的乐趣，大家都接受Ruby的设计哲学，用同一种思维方式来把不同的问题抽象成为代码，形成了现在这样一个积极向上的优秀社区。那么我想在Elixir身上我也看到了这种潜力，虽然是一门新生的语言，他继承了Erlang的所有优点，语法上融入了Ruby的人性化编程的精髓，也许在不久的将来会成为并发编程中非常热门的语言.



##学习资料

######Erlang
Erlang官网 [http://www.erlang.org/](http://www.erlang.org/) 

 
Programming Erlang [http://pragprog.com/book/jaerlang/programming-erlang](http://pragprog.com/book/jaerlang/programming-erlang)  
O'REILLY Erlang [http://shop.oreilly.com/product/9780596518189.do](http://shop.oreilly.com/product/9780596518189.do)  
Learn you some erlang(一本在线电子书) [http://learnyousomeerlang.com/](http://learnyousomeerlang.com/)


######Elixir
Elixir官网 [http://elixir-lang.org/](http://elixir-lang.org/)  
Programming Elixir [http://pragprog.com/titles/elixir](http://pragprog.com/titles/elixir)
