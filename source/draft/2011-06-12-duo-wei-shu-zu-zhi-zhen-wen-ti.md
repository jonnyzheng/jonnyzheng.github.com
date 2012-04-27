--- 
categories: 
  - 学习
comments: true
layout: post
published: true
status: publish
tags: 
  - c
title: 多维数组指针问题
type: post
---
一站式编程里的题目:

 

定义以下变量：


{% codeblock %}

char a[4][3][2] = {{{'a', 'b'}, {'c', 'd'}, {'e', 'f'}},
{{'g', 'h'}, {'i', 'j'}, {'k', 'l'}},
{{'m', 'n'}, {'o', 'p'}, {'q', 'r'}},
{{'s', 't'}, {'u', 'v'}, {'w', 'x'}}};

char (*pa)[2] = &a[1][0];
char (*ppa)[3][2] = &a[1];

[/cc]

要想通过pa 或ppa 访问数组a 中的'r' 元素，分别应该怎么写？

 

 

 

代码如下：
[cc lang="c"]
#include <stdio.h>

int main(void)
{
char a[4][3][2] = {{{'a', 'b'}, {'c', 'd'}, {'e', 'f'}},
{{'g', 'h'}, {'i', 'j'}, {'k', 'l'}},
{{'m', 'n'}, {'o', 'p'}, {'q', 'r'}},
{{'s', 't'}, {'u', 'v'}, {'w', 'x'}}};

char (*pa)[2] = &a[1][0];
char (*ppa)[3][2] = &a[1];
pa=pa+5;
ppa++;
printf("pa now is %c\n",pa[0][1]);
printf("ppa now is %c\n",ppa[0][2][1]);

return 0;
}
[/cc]

 

解释:

 

1.首先说
[cc lang="c"]
char (*pa)[2] = &a[1][0];
[/cc]
char (*pa)[2] 意思是 含有两个类型是char元素的数组的数组指针，&a[1][0]则代表 数组a中第二行第一组的首地址，赋值后pa将地址指向了 {'g','h'} 数组，那么如果要到{q,r}我们只要指针的地址指到{q,r}数组的首地址就可以了，将pa+5移动了指针指向的地址。

 

最后打印r
[cc lang="c"]printf("pa now is %c\n",pa[0][1]);[/cc]
pa[0][1] 表示访问这个数组中的第二个元素，也就是r

 

 

2.
[cc lang="c"]char (*ppa)[3][2] = &a[1];[/cc]
ppa代表含有一个二维数组的数组指针，目前指向了第二行
[cc lang="c"]{{'g', 'h'}, {'i', 'j'}, {'k', 'l'}}[/cc]
要想访问到必须再向下一行： ppa++

最后访问r
[cc lang="c"]printf("ppa now is %c\n",ppa[0][2][1]);
{% endcodeblock %}


 

其实也可以不用挪动指针而直接访问就可以，比如pa,  pa[5][1]就是r,  paa[1][2][1]也可以访问到r.

 
