--- 
categories: 
  - 杂货铺
comments: true
layout: post
published: true
status: publish
tags: 
  - git
title: "Git Rebase 后本地commit记录丢失"
type: post
---
今天下班前要代码要上测试环境，和另外两个同事一起merge代码，等我merge的时候我发现pull不到更新，于是问他俩，他们竟然都签入到了master里去了，邮件里写的很清楚要签入到分支的，两个人一个都没看邮件，当时不hold住就要吐血了。

OK,  反正可以用rebase,  于是用TortoiseGit的rebase功能把他们在master上的代码merge到当前分支上来，一切搞定后提交测试，结果他俩的代码都正确了，我之前写的功能没有了，再看看代码今天写的东西都没有了，shit, 看来真是hold不住了。

查看log记录，没有push到远程的commit也都不见，难道真的要重写一遍么，还好万能的 Google 救了我。

即使rebase命令已经干掉了上一次push到现在的所有提交记录但是在本地文件里还是有相关信息的，找到项目目录下的git文件 .git\logs\refs\heads\branchname, 打开来看看


{% codeblock %}
0000000000000000000000000000000000000000 f7f3b11f66b709dcd93e1c15825270dce90adc5e jonny <jonny.zheng@lexisnexis.com> 1314676889 +0800	branch: Created from HEAD

f7f3b11f66b709dcd93e1c15825270dce90adc5e 74173c185de3ce5ee846bc7e0326481276372071 jonny <jonny.zheng@lexisnexis.com> 1314682422 +0800	commit (amend): init

74173c185de3ce5ee846bc7e0326481276372071 93ba66f74b9e11ed8acae3e6ddd1159be1ff9ef4 jonny <jonny.zheng@lexisnexis.com> 1314682760 +0800	commit (merge): Merge branch 'b3.3' of 10.123.4.212:/opt/git/jolnet into b3.3

93ba66f74b9e11ed8acae3e6ddd1159be1ff9ef4 9273c7dbd497d070ebd8704f284444f658666954 jonny <jonny.zheng@lexisnexis.com> 1314772033 +0800	commit: 1

9273c7dbd497d070ebd8704f284444f658666954 9530c1ca1730ab1c249d3c301817bd037b3f6773 jonny <jonny.zheng@lexisnexis.com> 1314786503 +0800	commit: daily check in

9530c1ca1730ab1c249d3c301817bd037b3f6773 c91bb413528815321b10929d9cd26480de64f3bf jonny <jonny.zheng@lexisnexis.com> 1314787003 +0800	commit (amend): daily check in

c91bb413528815321b10929d9cd26480de64f3bf 3006dced1c8d39108483927bd5cc1e9ee4d1ec05 jonny <jonny.zheng@lexisnexis.com> 1314788056 +0800	3006dced1c8d39108483927bd5cc1e9ee4d1ec05: updating HEAD

3006dced1c8d39108483927bd5cc1e9ee4d1ec05 813acf5e60bc0d2e64c9682fe7498dc79606e1e1 jonny <jonny.zheng@lexisnexis.com> 1314788446 +0800	813acf5e60bc0d2e64c9682fe7498dc79606e1e1: updating HEAD

813acf5e60bc0d2e64c9682fe7498dc79606e1e1 ada2680266b192a0ebf8d4a43ca8ca4812aadc78 jonny <jonny.zheng@lexisnexis.com> 1314788695 +0800	ada2680266b192a0ebf8d4a43ca8ca4812aadc78: updating HEAD

ada2680266b192a0ebf8d4a43ca8ca4812aadc78 a259bcff43cc726be011efbbeb5975ffafb87ee5 jonny <jonny.zheng@lexisnexis.com> 1314788871 +0800	commit: rebase

{% endcodeblock %}

最后一行是rebase的操作，我只要新建个分支在里面找回前一行commit的代码就可以了。


{% codeblock %}
git checkout -b my_recovery 0800	ada2680266b192a0ebf8d4a43ca8ca4812aadc78
{% endcodeblock %}

方法2（这个更简单）
{% codeblock %}
git reflog
# Suppose the old commit was HEAD@{5} in the ref log
git reset --hard HEAD@{5}
{% endcodeblock %}


终于找回了丢失的代码，可以不用重写一遍了，而且学到了很有用的解决方法，看来犯错也不都是坏事。

最后补一句，还是命令行比较好用，图形化的复杂操作不会用啊。
