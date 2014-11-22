---
layout: post
title: "(译)第二部分：什么是merge?"
date: 2014-10-15 00:03
comments: true
categories: git
tags: [merge]
---

####内容提要

* 第一部分：[commit hash是什么？](http://www.fusijie.com/blog/2014/10/08/what-is-a-commit-hash/)
* 第二部分：merge是什么？
* 第三部分：[rebase是什么？](http://www.fusijie.com/blog/2014/11/18/what-is-a-rebase/)

在[第一部分](http://www.fusijie.com/blog/2014/10/08/what-is-a-commit-hash/)我们创建了一个小的demo仓库，它拥有着一个`feature1`分支，而且这个分支已经准备好要merge到`master`分支中了。

<!-- more -->

![](http://i.imgur.com/kxDIgKl.png)

此时，我们可以选择merge或者rebase `feature1`分支到`master`分支。关于rebase将会在[第三部分](http://www.fusijie.com/blog/2014/11/18/what-is-a-rebase/)进行介绍。现在我们来看一下，采用merge的方式到底发生了什么。把分支合并到一起是非常直接的。首先需要将切换到要合并进去的分支，在这里，因为我们要将`feature1`合并到`master`分支，所以需要切换到`master`分支。

![](http://i.imgur.com/S0av3NM.png)

我切换到`master`分支，然后将`feature1`分支合并进去。回过头来再看一下这之中发生了什么，为什么Source Tree生成的图形是这个样子的。

还记得[第一部分](http://www.fusijie.com/blog/2014/10/08/what-is-a-commit-hash/)中`Commit 3`和`Commit 4`引用着同一个先前commit吗？`Commit 2`是这两个commit共同的祖先，因为`Commit 3`是在另一个分支上创建的，而`Commit 4`是在`master`分支上创建的，所以它完全不知道`Commit 3`的存在。在`feature1`上我们添加了更多的commit。`Commit 5`直接引用了`Commit 3`，因为`Commit 4`只在`master`分支上有效，`Commit 6`直接引用了`Commit 5`。

当我们将`feature1`合并到`master`中，它并不是通过某种方式神奇地把这些commit都移到`master`分支上。实际上，它创建了一个包含了`feature1`分支上**所有的**变更的全新commit。这个commit叫`Merge branch 'feature1'`，就像这样：

![](http://i.imgur.com/RECAHy7.png)

如果你注意到上图中的commit差异，就会看到我添加到`index.txt`中二了吧唧的这几行。你应该会注意到这几行是通过各个commit分开地添加进去的。然而，现在你看到的是所有的这些改变都在单一的一个差异中。

Git所做的只是把`feature1`中所有的commit的所有差异汇聚到一个单一的commit中。这个新的commit干了一些我们之前没有讨论过的事。从上图可以看到它拥有2个祖先，也就拥有着从`Commit 4`和`Commit 6`过来的两条线。为什么呢？commit可以保存多个先前commit的索引。我现在才来讲这个话题是因为我不想太早地引起混淆。

当一个commit被创建的时候，它所引用的之前commit数量可以是一个，多个，甚至没有。通常只有仓库中第一个commit才会没有先前commit，而merge commit一般都拥有超过一个的先前commit。

如果你还记得[第一部分](http://www.fusijie.com/blog/2014/10/08/what-is-a-commit-hash/)的话，分支，其实实际上只是一个指向一个指定commit的指针而已。

![](http://i.imgur.com/S0av3NM.png)

你可能会注意到`feature1`仍然指向了`Commit 6`，而`master`分支指向了新的merge commit，很简单，因为我们是将`feature1` 合并到`master`。如果我们将分支切换到`feature1`，然后再把`master`合并进来，那么Git所做的就是一个*fast-forward* marge（快进合并），这会把`feature1`的指针指向最新的commit。

![](http://i.imgur.com/Ggvb3UK.png)

如果我们完全删除了`feature1`分支，你可能会以为粉色的线消失，但是你错了。

![](http://i.imgur.com/rcSSPFa.png)

记住，Source Tree和其他的Git可视工具是通过遍历你的commit，用索引的commit hash连接各个commit来生成图形的。分支只是一个指向指定commit的指针。当你从一个远程仓库拉取更新（pull）时，Git所做的是：

* 1.下载所有你本地机器上没有的commit
* 2.合并丢失的commit到你的本地仓库，或是通过一个merge commit，或是通过一个*fast-forward* merge，前提是你在最后一次拉取更新后没有做任何的修改。
* 3.把你的本地分支指向最新的commit。

如果你曾经混淆过`master`和`origin/master`指针，那现在你应该知道它们是是啥了。`origin/master`告诉你你的`origin`远程`master`分支指向哪。如果我给这个demo仓库添加了一个远程仓库叫`origin`，然后在本地仓库上做了一些commit，Git的历史可能会像这样：

![](http://i.imgur.com/hSizNJB.png)

你会看到`master`分支指向了最新的commit，而`origin/master`指向了前一个merge commit。Source Tree甚至提示我们说有一个commit可以推送（push）到远程仓库。如果我们推送上去，Git将会上传丢失的commit，然后更新你的远程分支指针，此时`origin/master`已经和你的本地`master`分支指向了相同的commit。

![](http://i.imgur.com/pmyLiFb.png)

希望你现在对Git的合并功能有了更好的理解。跳到[第三部分]()让我们深究下rebase，看看它和merge有什么区别吧唧。

>英文地址:[http://codetunnel.com/merge-vs-rebase-part-2-what-is-a-merge/](http://codetunnel.com/merge-vs-rebase-part-2-what-is-a-merge/)
