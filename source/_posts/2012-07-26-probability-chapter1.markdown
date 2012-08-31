---
layout: post
title: "probability chapter1"
date: 2012-07-26 10:55
published: false
status: draft
tags: []
comments: true
type: post
categories: math 
---

### 第一章 排列组合 permutation

#### How many different letter arrangements can be formed using the letter  PEPPER?
  
首先一共有 `6!` 种可能，但是有3个P，2个E， 他们之间是会重复的,重复的可能有3!2!种.
去除掉重复的 6!/(3!2!) = 60

类似这种重复的问题可以用公式： 
$$\frac{n!}{(n_1!n_2!\dots n_n!)}$$


#### How many different groups of 3 could be selected from the 5 items A,B,C,D,E

选3个一共有5\*4\*3 种，但是里面会有重复的，比如 ABC, ACB, BAC, BCA ... , 一共有6种，结果是：
$$\frac{5·4·3}{3·2·1}=10$$, 总结出公式: $$\frac{n(n-1)\dots(n-r+1)}{r!}=\frac{n!}{(n-r)!r!}$$ 

我们定义从n个元素里选出r个(n ≥ r)的不同组合方式表示: $$\binom{n}{r}$$ , 完整公式就有了:

$$
\begin{equation*}
  \binom{n}{r} = \frac{n!}{(n-r)!r!}
\end{equation*}
$$



### The binomial theorem(二项式定理)

$$
\begin{equation*}
 (x+y)^n = \sum_{k=0}^n\binom{n}{k}x^k y^{n-k}
\end{equation*}
$$


#### 求n各元素中可以选出多少子集
解： 假设n = 3，一个元素的子集有 {1},{2},{3} 。2个元素的有{1,2} ,{2,3}, {1,3}。 3个元素的子集只有一个{1,2,3}。
所以就相当于: $$ \binom{3}{1}+\binom{3}{2}+\binom{3}{3} = \sum_{k=0}^{n=3}\binom{n}{k}$$, 所以由公式可以得到：

$$
\begin{equation*}
  \sum_{k=0}^n\binom{n}{k} =  \sum_{k=0}^n\binom{n}{k}1^k 1^{n-k} = (1+1)^n = 2^n
\end{equation*}
$$

最后由于选0个元素的子集是没有意义的，所以还要减去1种可能，结果就是 $2^n -1$

### 多系数组合

#### 公式

$$
\begin{equation*}
  \binom{n}{n_1,n_2,\dots,n_r} =  \frac{n!}{n_1!,n_2!,\dots,n_r!}
\end{equation*}
$$




#### 有一个警察局有10个警察，其中5个需要巡逻，2个呆在办公室里，3个做后备，一共有多少种组合

$$
\begin{equation*}
	\frac{10!}{5!2!3!} = 2520
\end{equation*}
$$



#### 10个小孩需要被分成两组打篮球，一共有多少种组合？

$$
\begin{equation*}
	\frac{10!/(5!5!)}{2!} = 126
\end{equation*}
$$

这里的情况和上面不一样，因为是平均分不存在特定的组，比如上面5个人的组就是特定的组，和3个人的组是不一样的。
所以这里少了不同组的可能。


### 多项式公式

$$
\begin{equation*}
 (x_1+x_2+\dots+x_r)^n = \sum_{(n_1,\dots,n_r)}\binom{n}{n_1,n_2,\dots,n_r}x_1^{n_1}x_2^{n_2}\dots x_r^{n_r}
\end{equation*}
$$


