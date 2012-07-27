--- 
categories: []
comments: true
layout: post
published: false
status: draft
tags: []
title: "git 团队开发实践"
type: post
---

##初衷
由于项目团队在使用git的过程中碰到了很多问题,我经常在网上查找git的相关文档，尽管好的资料也很多，但是大多是按照对git命令用法介绍的思路进行组织的，
所以需要开发人员对git本身从理念到命令达到一定程度的熟练后才知道如何灵活应用，学习曲线比较高，而且团队成员水平和兴趣又各不相同，让他们自己学习似乎需要
很长的时间，效果还不一定好，于是我整理不少git的相关资料，希望从团队开发实践的角度出发尽量使用易于理解的语句将所需要的git相关知识写下来，给刚刚接触git的
同学提供一个完整的开发实践,供大家参考。

##理解git

git 是什么大家应该很清楚了，下面的链接里有git基本的相关介绍和git与其他版本控制工具之间的比较。

##### git相关教程
* [使用 Git 管理源代码](http://www.ibm.com/developerworks/cn/linux/l-git/)
* [git-scm](http://git-scm.com/)
* [pro git](http://progit.org/book/)

#####git和其他scm工具的比较
* [Why Git is Better than X](http://zh-tw.whygitisbetterthanx.com/)
* [版本控制系统的选择之路](http://www.oschina.net/question/84514_9508)

#####初学者需要理解的是git与cvs,svn有很大的不同:  

* 在团队开发中git一般分为远程库和本地库两个概念,远程库是整个团队的代码的核心,本地库是程序员自己的git库。
程序员的commit, 创建分支等操作都是在他自己的库里折腾,和远程库一点关系也没有,只有在使用git push才会将改动的代码或新创建的分支上传到远程库中,
使用git pull将远程库里的代码同步到本地库来。
这样的分布式构架给开发带来了相当大的便利，你不需要和远程库连接就可以管理自己修改的代码和分支，当你真正把某一个功能开发完毕的时侯再把需要提交的版本push到远程库里。

* git没有为个体文件建立版本的说法，每次的提交操作都是对当前项目的完整源代码的一个新版本，所以当你checkout某个版本的时侯是整个项目代码切换到了那次提交时代码的状态。

##开始项目

为了便于大家理解我们来虚构一个项目，什么是这个世界上永恒的话题 --> 异性, 是的，那么我们就搞个教大家如何把妹子的网站好了, 同时我们有了4个自愿的开发者（jonny,jack,nick,peter），
我们励志于拯救正处在水深火热中的单身男程序员们。 

###搭建git环境


###分支流程设计

由于git的分支管理相当的灵活，所以没有一个统一的流程和规范的话会比较混乱，而且也发挥不出git分支的优势。下面介绍一下分支的管理流程，大家可以根据自己团队和项目的具体
情况有选择性的使用。

先介绍一下在网上广为流传的一篇文章，这里有英文原文和翻译的版本：


这里把分支分为master和develop两大分支，master用来存放每个稳定的版本，每次从master取到的版本都是最后的上线版本，平时的开发任务则在develop分支上，基于develop分支，
我们又可以根据不同的任务，再在上面开分支。比如：Feature, bug, release version。

如果一个团队的release都是比较走流程的话我们可以在develop分支上开一个release version版本分支，

git checkout develop
git checkout -b v0.3

这样我们所有的release 相关开发都在v0.3这个分支下进行开发，同时




*	删除git上的文件并且保留本地文件 git rm file --cached
*   添加tags到远程库 git push origin --tags
*   获取远程分支 git fetch


###git stash命令
当写代码的时候需要切换到别的分支又不想commit还没有完成的代码的时候我们可以用git stash命令，stash命令相当于一个暂存区域，我们可以把当前的改动通通放到这个区域里面去。


```
$ git stash save 
Saved "WIP on master: e71813e..."
```

stash区域可以存入1个以上的工作状态，用git stash list命令就能看到所有在stash区域里的工作状态

```
$ git stash save 
Saved "WIP on master: e71813e..."
```


从上面的list里面我们可以看到一共有两个item, stash@{0}和stash@{1},数字用来标记当前的位置，stash使用的是栈模式，先进后出，所以当你已经完成了当前工作，想把之前存入的改动找回来的时候可以用我们可以用pop弹出一个item或者用apply 接受一个改动。需要注意的是git stash pop 永远弹出的是stash@{0}, 而apply可以使用任何项，还有就是pop弹出的项不再出现在list里面， 具体用法如下:


```
$ git stash save 
Saved "WIP on master: e71813e..."
```

从上面的命令可以看出pop之后stash里的item数量从2个变成了1个。

当stash里的数据不再需要的时候我们用git stash clear命令清空整个空间。
#### 用pull好还是用rebase好

在远程分支已经有他人提交了最新的分支时我们往往需要pull得到最新的代码, git pull = git fetch + git merge。一方面合并会产生除了合并双方（或多方）所有提交外的一个新提交，增加了代码审核的负担，另一方面本地多个提交混杂一起与远程分支合并会更困难

http://www.worldhello.net/gotgithub/04-work-with-others/020-shared-repo.html
http://www.cnblogs.com/iammatthew/archive/2011/12/06/2277383.html
